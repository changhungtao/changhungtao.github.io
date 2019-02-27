---
title: "Test EN. Language"
subtitle: "Staticman Powered GitLab Pages"
date: 2018-09-16T19:12:15+02:00
type: post
categories:
- blogging
tags:
- GitLab
- Staticman
draft: false
---

I put some "why" questions here so as to keep focus on the technical setup of
the GitLab repo and the optional Staticman API server.

### Why static blogs instead of dynamic ones?

- quicker loading time
- better reliability (can handle more request)
- no database needed
- greater control on content, styles and layout

### Why static comments?

- allow feedback from visitors
- site owner _owns_ the comment locally (unlike WordPress, Facebook, Disqus, etc)
    + no remote database needed, so no need to worry server errors from
    third-party commenting services.
    + greater control over the _rendering_ of the comments (allow additional
    features such as Markdown syntax, and $\KaTeX$  support)
    + more accessible since static comments are _incorporated_ as HTML elements
    into the post.  _No_ JavaScript is required to retrieve the comments,
    contrary to most third-party commenting services.

Before [Staticman]'s deployment, another commenting system called [Pecosys] was
already available.  However, it's less convenient to handle visitor's requests
as emails.

The site [IRZ][IRZ] (in French) has been migrated from WordPress to
Jekyll+Staticman for two main reasons: save time and money.

- _each dynamic page took 10&nbsp;s_ to load, and he/she had
_3500&nbsp;articles_! vs faster static pages
- _annual hosting cost of 43€_ for WP vs _free_ GitHub Pages hosting

### Why GitLab instead of GitHub?

- GitLab CE, as a _software_, is _open source_, whereas GitHub is _proprietary_.
- GitLab has _native_ CI/CD support, which enables _automated_ package builds on
the remote server.
- GitLab, as a remote _Git host_, allows _unlimited private repo_ for _every_
user; GitHub allows this for _paid_ users only.

### Why is the significance of Staticman's GitLab support?

In the past, [Staticman] was known to support GitHub _only_.  As a result, it's
_not_ entirely _independent_ of proprietary technologies.  As one sees in
[issue&nbsp;#22][issue22], such technical restraint stops users from adopting
this great commenting system.

With [pull request #219][pr219], [Staticman&nbsp;v3][staticman3] is now capable
of shipping HTML form data to GitLab in the name of a GitLab bot.  Since GitLab
community edition (CE) is _open source_, it is, _in principle_, possible to run
an instance of [Staticman] depending _entirely_ on free software.

### Is Staticman secure?

Some users are worried about the security of [Staticman].

1. _With user's authorization_, it collects only HTML form data and sends a
_merge request_ containing the form data in YML format to user's repo.  _User's
approval is required_ as long as `moderation: true` in `staticman.yml`.
2. One can set integrate third-party services like Akismet and reCAPTCHA to stop
bots and spam.
3. Critical secrets (e.g. OAuth App secret) in config files are encrypted with
RSA by a GET request to the `/encrypt` endpoint, which only do the math and
return the result.
4. Under [Staticman]'s open authentication (OAuth) flow, users' secrets are
_never_ stored nor leaked.  The _entire_ project is open source, meaning that
_anyone_ can verify the code.
    1. Potential commentators are redirected to GitLab's site, who actually gets
    the passwords from them.
    2. In order to post comments, they have to grant the OAuth app some rights
    (`read_user` and `api`) for legitimate reasons:
        - `read_user`: enable the bot to retrieve GitLab user profile
        - `api`: for accessing GitLab user profile through API
    3. The OAuth app then responds with a redirect URI, which
        - points back to the static site, and
        - contains a `code` parameter at the end of the URL.
    4. The static site sends a GET request to `/encrypt` `code` as `oauth-token`.
    5. The static site includes the `oauth-token` in the form data.
5. Git repo owner has _total_ control over the comments just like the posts.
He/she can remove whatever he/she dislikes.

Since the `oauth-token` _never_ expires, _client-side_ browser settings need to 

N.B.: In this section, "[Staticman]" refers to the _Node.js app_ (available for
download on GitHub) rather than the [_public API instance_][api], which is
exclusively accessible to the package owner himself.  In case of concerns over
the server, I suggest [running your own API instance](../2) to ensure 
freedom.

### Why did I write my own guide?

Simple reason: I _couldn't_ find a simple and comprehensible guide for users who
_don't_ know [Node.js][nodejs].  Therefore, I decided to write my own guide
_targeting at end users_ to give them a general idea about

1. [the _client-side_ repo setup](../1)
2. [the _server-side_ Staticman&nbsp;v3 API setup](../2)

<i class="fas fa-info-circle fa-fw" aria-hidden></i> Until now, [Staticman]'s
project owner, who has set up the [public development instance][staticman3] of
Staticman&nbsp;v3 API server _didn't_ inform users of the details of the GitLab
bot.  [Nicholas Tsim][ntsim], who developed [Staticman's GitLab support][pr219],
simply said "it can be any user".  However, after setting up my own user, I
encountered difficulties in understanding the _whole_ setup, especially the role
played by the GitLab bot and its _raison d'être_.

Moreover, unlike the official documentation on [Staticman]'s site,
[Nicholas Tsim's setup guide][dev_setup] has mixed up GitLab repo config with
API server config in one section.  On one hand, this serves as an informative
first-hand source of information for other _developers_.  On the other hand,
_users_ might find it a bit difficult to get an overall idea about the whole
setup.

Despite the errors encountered while I was following his guide, _I was indebted
to his assistance_ over these few weeks.

[nodejs]: https://nodejs.org/
[Staticman]: https://staticman.net/
[staticman3]: https://dev.staticman.net/
[ntsim]: https://github.com/ntsim
[pr219]: https://github.com/eduardoboucas/staticman/pull/219
[issue22]: https://github.com/eduardoboucas/staticman/issues/22
[dev_setup]: https://github.com/eduardoboucas/staticman/pull/219#issuecomment-417857360
[Pecosys]: https://blogduyax.madyanne.fr/2017/un-blog-plus-statique/
[IRZ]: https://irz.fr/wordpress-jekyll
[api]: https://api.staticman.net