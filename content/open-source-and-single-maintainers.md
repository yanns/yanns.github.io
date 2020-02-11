+++
title = "Open source and single maintainers"
date = 2020-02-11
path = "blog/2020/02/11/open-source-and-single-maintainers/"
+++

# Disparition of a colleague

Last year I was in the very sad situation of [losing one colleague and friend, Oleg](https://github.com/sangria-graphql/sangria/issues/445).

He was the creator and the maintainer of some open source libraries.

The most famous one, [Sangria](https://sangria-graphql.org/), is a scala implementation of GraphQL. I know that this library is used by several companies, like Twitter or the New York Times.

Oleg put a lot of time and energy into that library, talking about it at [different conferences](https://www.youtube.com/watch?v=ymILgZAdfnA).

After his disparition, we, his colleagues, checked on [how to proceed with Sangria in the future](https://github.com/sangria-graphql/sangria/issues/446).

And I can tell that this is a very painful process.

Oleg was the only one granted for almost all repositories, hosted on Github.

## Dealing with Github

The Github process for dealing with that situation is very fuzzy. At each step, we got asked for more papers than at the previous step.

I don't want to go into details. But the issue has been open for months, and we still don't have the rights.

## The temporary solution

[As a temporary solution, we set up another Github organization, and we had to fork all repositories](https://github.com/sangria-graphql/sangria/issues/446#issuecomment-546281588).

For this, I was lucky that [Travis Brown](https://twitter.com/travisbrown) jumped in and set up the initial infrastructure.

## Dealing with Sonatype

Sonatype was very supportive.

They were responsive and could give us the [rights to publish the artifacts of Sangria](https://issues.sonatype.org/browse/OSSRH-48782).

With those permissions, we could [release new versions](https://github.com/sangria-graphql-org/sangria/releases).

Thx again Sonatype for the support!

# Why am I telling you that

I don't tell you that to blame Github. Any company has processes and internal friction.

If it's not with Github, it could have been another company we would have issues with.

What I'd like to emphasize is how complex and time consuming it is to handle an open source library when the maintainer has disappeared.

I don't wish anyone to have the same tragic destiny as Oleg. But there are also other reasons why one cannot and don't want to maintain an open source library anymore.

All this process would have been much more simple if we would have set it up while Oleg was alive.

# About me being a single point of failure

With that in mind, [I looked for other maintainer for a modest library](https://github.com/leanovate/play-mockws/issues/66)

And I found [someone](https://github.com/avdv) who helped me maintaining the library, reviewing and merging pull requests.

Lastly we also organized that he could publish artifacts.

And he took care of [publishing the last release](https://github.com/leanovate/play-mockws/releases/tag/v2.8.0).

I'd like to thank him for jumping in. I'm now feeling very good not being alone on that project.

# Avoid being a single point of failure

That would be my main message: avoid being a single point of failure.

Think about other people using your libraries.

Onboard other maintainers while you can do it.

I understand that the administrative work that is needed to grant access to a new maintainer is not a very motivating task.

But please do it. It will make everyone's life much easier.
