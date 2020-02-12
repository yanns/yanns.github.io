+++
title = "Open source and sole maintainers"
date = 2020-02-11
path = "blog/2020/02/11/open-source-and-single-maintainers/"
+++

# Open source and sole maintainers

Last year I was in the very sad situation of [losing one colleague and friend, Oleg](https://github.com/sangria-graphql/sangria/issues/445).

He was the creator and the maintainer of some open source libraries.

Among them the famous [Sangria](https://sangria-graphql.org/), a scala implementation of GraphQL. This library is used by major companies, such as Twitter or the New York Times.

Over the years Oleg has put a lot of time and energy into building and maintaining Sangria. He also gave many talks about it at [different conferences](https://www.youtube.com/watch?v=ymILgZAdfnA).

After his disparition, we, his colleagues, started investigating [how to proceed with Sangria in the future](https://github.com/sangria-graphql/sangria/issues/446).

It become apparent early that the bureaucracy involved will be very painful process. Partly due to the fact that Oleg was the only one who granted himself access to almost all his repositories hosted on Github.

## Dealing with Github

The general procedure of dealing with GitHub during this process of transitioning the ownership is very fuzzy. It involves a lot of paperwork. At each subsequent step, we were asked for more documents than during the previous one.

I will spare us all the details as I would like to focus on the current status. By now the process has been ongoing for several months. Sadly, we still do not have transitioned ownership rights of Sangria.

## The temporary solution

As a temporary solution, we [set up another Github organization](https://github.com/sangria-graphql/sangria/issues/446#issuecomment-546281588), and we had to fork all repositories.

For this I would like to thank [Travis Brown](https://twitter.com/travisbrown) who jumped in and set up the initial infrastructure.

## Dealing with Sonatype

Also Sonatype was very supportive. They were responsive and helped by granting us the [rights to publish the artifacts of Sangria](https://issues.sonatype.org/browse/OSSRH-48782). With these permissions, we were able to [release new versions](https://github.com/sangria-graphql-org/sangria/releases).

Thanks again Sonatype for the support!

# Why am I telling you that

The point of my writings is not to blame Github! Any company follows processes in place and struggles with cases like this. If it was not Github today, it would have been another company we would be involved with.

What I would however like to emphasize is how complex and time consuming it is to work with open source library when the maintainer has disappeared.

Oleg’s tragic destiny is not the only cause why one cannot or do not want to maintain an open source library anymore. Many other valid reasons exist.

However, if we would have prepared ourselves by setting up a few things while Oleg was still alive, the remaining steps needed would have caused far less problems and friction for all of us.

# About me being a single point of failure

Having said all that, [I looked for another maintainer for a modest library of mine](https://github.com/leanovate/play-mockws/issues/66) and found [someone](https://github.com/avdv) who stepped up for the library by reviewing and merging pull requests.

Additionally, we also made sure that he can publish artifacts. As a result he took care of [publishing the last release](https://github.com/leanovate/play-mockws/releases/tag/v2.8.0).

I would like to thank him for jumping in. I am now feeling relieved by not being alone on this project anymore.

# Avoid being a single point of failure

My main message here is: try to make sure you are not a single point of failure for your projects. Think of others using your libraries and their expectations and dependencies towards it. Onboard other maintainers early if you can and delegate some of your reponsibilities.

Admittedly administrative work is needed to grant access to a new maintainer and align on the resulting expectations. Frankly, it is not a very motivating task.

Still, please go through it and do it.

It will make everyone’s life much easier.
