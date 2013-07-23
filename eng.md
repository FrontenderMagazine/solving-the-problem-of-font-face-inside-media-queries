# Solving the Problem of @font-face Inside Media Queries

Mobile data connections frequently aren’t as good as desktop ones, therefore the
download time of custom fonts can take up to several seconds. A good approach to
solving this is to use the `@font-face` rule only for a certain range of media
and screens using media queries. Unfortunately this approach doesn’t work for
some browsers including all versions of Internet Explorer and FireFox 10 and
lower. This article discusses a solution for this problem that balances
performances and hacks.

## What’s the Problem?

When I started creating my new website, I wanted to create a strongly mobile
optimized design while also keeping in mind performance. One of the things I
really wanted to avoid was the effect, frequently seen on websites when accessed
via a smartphone, of a white block where there is supposed to be the content.
This issue is caused by a custom font applied to the text that it isn’t
completely loaded. This issues is illustrated in the image below.

![screenshot][white block where there is supposed to be the content]

So, what’s the solution? As mentioned in the introduction, a good approach is to
apply a custom font, using the `@font-face` rule only for a specific range of
media and screens using media queries. You might think that the best solution
would be to apply the custom font only to those users visiting the website with
a fast connection, and you’re absolutely right. Unfortunately there isn’t yet
such a system, although it [has been discussed][1] several times on the
[WHATWG][2] mailing list.

## @font-face inside a @media query

To better explain the solution, we’ll create two files: index.html and
style.css. The first is the ideal page having the text with the custom font
applied, while the latter contains the style of the page.

Our hypothetical index.html page could be the following:

    <!DOCTYPE html>
    <html>
       <head>
          <link rel="stylesheet" href="style.css" />
       </head>
       <body>
          <p>
             This is an amazing text because has a custom font.
          </p>
       </body>
    </html>

As you can see, it’s very basic since it has only one paragraph and the link to
the stylesheet.

The following is our hypothetical style.css file using a mobile first approach:

    p {
       font-family: Arial,Helvetica,sans-serif;
    }

    @media only screen and (min-width: 980px) {
       @font-face {
          font-family: "OctinSports";
          src: url("fonts/octinsports.eot");
          src: url("fonts/octinsports.eot?#iefix") format("embedded-opentype"),
          url("fonts/octinsports.woff") format("woff"),
          url("fonts/octinsports.ttf") format("truetype"),
          url("fonts/octinsports.svg#OctinSports") format("svg");
          font-weight: normal;
          font-style: normal;
       }

       p {
          font-family: "OctinSports";
       }
    }

The stylesheet sets an Arial font for all the paragraphs, with several
fallbacks. The font stack used is composed of fonts that are really common and
highly present in the major operating systems. In fact, based on
[CSSFontStack][3], Arial achieves a 99.84% on Windows and the 98.74% on Mac. In
case none of the selected fonts is available, the CSS specifies a generic sans-
serif font.

The second part of style.css defines the media query. With it, we’re targeting
all the devices that have a screen with a width of at least 980px (you can use a
different breakpoint). Inside it, we’ve set the @font-face rule to download the
custom font, using the [Fontspring Bulletproof @Font-Face Syntax][4]. Then,
we’re simply applying the new font to all the paragraphs, overriding the
previously set style.

## Browser Compatibility

While this solution seems really smart because it applies a custom font only for
users that have a big screen and should, theoretically, not have problem for a
little bandwidth overhead, unfortunately it is not supported by Internet
Explorer 10 and lower, as well as [Firefox 10 and lower][5]. Although you may
think it’s a strange behavior or bug, it actually comes from [the CSS2.1
specifications][6] that asserts that at-rules inside @media are invalid in
CSS2.1.

So, while the last version of Internet Explorer still doesn’t support this
feature, the company behind FireFox, Mozilla, [added the support][7] for
versions after the 10th. Therefore, the issue is really significant only for
Internet Explorer. Another point to keep in mind is also that Internet Explorer
8 and lower doesn’t understand media queries, so even if they update new
versions, the problem remains for older ones.

## How to (Partially) Solve the Problem

Is there a way to solve the problem of `@font-face` inside of a media query in
Internet Explorer? The answer is yes, partially.

As I said in the introduction, I came across this issue during the development
of my website. I wasn’t aware of the compatibility problem of Internet Explorer,
so, just like every good developer, the first thing I did to understand what was
going on, was to use Google. During my research, I found an article titled [Do
Not Put @font-face Inside @media Query][8] that gave the answer needed on the
cause, but not on the solution. Unable to locate a solution, I decided to create
one by myself.

## Solve the Problem for Internet Explorer 9 and Lower

Based on what I described so far, the problem is relevant only for Internet
Explorer, so we can try to target this specific browser. The first thing that
could came to mind is to create a separate CSS file, for example fonts.css,
containing all the `@font-face` rules, not in a media query, and use a
conditional comment. Thus, the source of fonts.css would look like this:

    @font-face {
       font-family: "OctinSports";
       src: url("fonts/octinsports.eot");
       src: url("fonts/octinsports.eot?#iefix") format("embedded-opentype"),
       url("fonts/octinsports.woff") format("woff"),
       url("fonts/octinsports.ttf") format("truetype"),
       url("fonts/octinsports.svg#OctinSports") format("svg");
       font-weight: normal;
       font-style: normal;
    }

Using this approach, we have the advantage of keeping the weight of the page the
same as before, apart for the few bytes for the conditional comment part, for
all the browsers but Internet Explorer. For IE, we’ll also be adding a little
overhead because of the additional stylesheet to download. However, as it
contains only a few lines specific to `@font-face` rules, after minimizing it,
an average file should cause an additional weight of no more than 1-2kb. In most
cases, this is an acceptable compromise.

The code that implements the approach, using the minified version, is the
following:

    <!--[if IE]>
       <link rel="stylesheet" href="css/fonts.min.css">
    <![endif]-->

This solution is simple but not definitive because, as you might know, [Internet
Explorer 10 dropped support for conditional comments][9], so this version will
ignore the snippet. So our problems aren’t finished yet.

## Solve the Problem for Internet Explorer 10

To target IE10 we’ll use a less clean approach, because we need a hack.
Specifically, the hack used is based on JavaScript and is discussed in the
article [IE10 CSS Hacks][10]. What this snippet does is to detect IE10 and test
if the window’s width is greater than or equal to 980px. If these conditions are
true, it adds the same fonts.min.css stylesheet to the `<head>` element of the
page. Remember that since it’s based on JavaScript, obviously, if it is disabled
the hack won’t work.

    <!--[if !IE]><!-->
       <script>
           if (Function('/*@cc_on return document.documentMode===10@*/')() && window.innerWidth >= 980) {
              var link  = document.createElement('link');
              link.rel  = 'stylesheet';
              link.href = 'css/fonts.min.css';
              document.getElementsByTagName('head')[0].appendChild(link);
           }
       </script>
    <!--<![endif]-->

## How this Solution works within a Responsive Web Design Approach?

A good question that you can arise is how this solution will work within the
responsive approach of a website. To answer this question, let’s split the
discussion into two branches. The first regards mobile users where we don’t want
the custom font, and the second regards desktop users where we do want the font
to be applied.

### Mobile Users

To start the discussion, let’s have a look at the [last statistics (June 2013)
about mobile browsers usage][11] provided by [StatCounter][12]. The link gives
us the following statistics:

1. Android: 29.06%
2. iPhone: 22.77%
3. Opera: 16.06%
4. UC Browser: 9.89%
5. Nokia: 7.38%
6. Chrome: 3.23%
7. BlackBerry: 3.11%
8. NetFront: 2.40%
9. iPod Touch: 2.21%
10. Others: 3.9%

An important important point to get from these statistics is that none of them
is Internet Explorer, and that its usage is lumped under “Others.” However,
thanks to [Craig Buckler][13] of SitePoint, I discovered that StatCounter
reports a usage of 1.4% for IEMobile (no distinction between versions).
Therefore, having the @font-face rule inside the @media query, we’re achieving
our goal of having a basic font at least for 98.6% mobile users.

Craig also wrote me that [NetMarketShare report shows 1.31% usage for IE9 and
1.0% for IE10 mobile][14]. Of course statistics can vary a little from site to
site but we get the general point.

Now, let’s take into account the two major versions of Internet Explorer for
mobile: 9 and 10.

As we’ve seen in a previous section, we’re targeting IE10 using a JavaScript
code that uses an hack and test for the window’s width. If the device has
JavaScript enabled, if the window’s width less than 980px, the fonts.min.css
stylesheet will not be added. Otherwise, if JavaScript is disabled, the snippet
won’t run and the custom font won’t be loaded or set.

About IE9, there isn’t much to do: the stylesheet will be loaded and the custom
font applied anyway. However, this issue should affect no more than the 50% of
those using IEMobile.

To wrap up, the font is not applied in roughly the 99.5% of the cases with only
IE9 failing. In my opinion, this is a big success.

### Desktop Users

As with the previous section, let’s have a look at the [last statistics (as of
June 2013) about desktop browsers usage][15]. The desktop statistics are the
following:

* Google Chrome: 42.71%
* Firefox: 20.01%
* Internet Explorer 10: 9.88%
* Safari: 8.39%
* Internet Explorer 8: 8.04%
* Internet Explorer 9: 6.79%
* Opera: 1.03%
* Internet Explorer 7: 0.49%
* Internet Explorer 6: 0.22%
* Others: 2.44%

Since we kept the `@font-face` rule inside the media query, we’re still
optimizing the website for all the browsers that do support both media queries
(not IE8 and lower) and `@font-face` inside media queries. As we’ve seen, these
are all the browsers but Internet Explorer. In conclusion, we’ve a proper
optimization for 74.58% users, good starting point.

For Internet Explorer 6-9, the custom font will be loaded, as we wanted, using a
conditional comment. So, to the previous percentage we can add another 15.54%
which lead to a partial total of 90.12%.

With regard to Internet Explorer 10, since we rely on a JavaScript method to
load the stylesheet, the custom font is loaded for all those who have JavaScript
enabled. I’m not able to provide any statistics on this but those with
JavaScript disabled should likely not exceed the 0.5%.

To wrap up, the font is applied in roughly the 99.5% of the cases. Another big
success.

## Conclusion

Custom fonts can give a nice look to a website and more and more developers and
designers are using them. However, we don’t have to forget to optimize for
mobile devices due to the increasing traffic coming from them. As we’ve seen in
this article, using it is possible to use a mobile first approach for web fonts
having `@font-face` inside media queries, together with the use of a couple of
tricks that target Internet Explorer. We can trust this technique because
they’re safe and let us achieve a really high success percentages: 99.5% both
for mobile users and desktop users.

[1]: http://lists.whatwg.org/htdig.cgi/whatwg-whatwg.org/2012-May/035799.html
[2]: http://www.whatwg.org/
[3]: http://cssfontstack.com/
[4]: http://www.fontspring.com/blog/the-new-bulletproof-font-face-syntax
[5]: https://bugzilla.mozilla.org/show_bug.cgi?id=567573
[6]: http://www.w3.org/TR/CSS21/media.html#at-media-rule
[7]: https://bugzilla.mozilla.org/show_bug.cgi?id=511909
[8]: http://robin.medvedi.eu/do-not-put-font-face-inside-media-query/
[9]: http://www.sitepoint.com/microsoft-drop-ie10-conditional-comments/
[10]: http://www.impressivewebs.com/ie10-css-hacks/
[11]: http://gs.statcounter.com/#mobile_browser-ww-monthly-201306-201306-bar
[12]: http://statcounter.com/
[13]: http://www.optimalworks.net/
[14]: http://marketshare.hitslink.com/browser-market-share.aspx?qprid=2&qpcustomd=1
[15]: http://gs.statcounter.com/#browser_version_partially_combined-ww-monthly-201306-201306-bar

[white block where there is supposed to be the content]: img/aeon-megazine-mobile.png