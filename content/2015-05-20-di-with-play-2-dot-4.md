+++
title = "DI with play 2.4"
date = 2015-05-20
path = "blog/2015/05/20/di-with-play-2-dot-4/"
+++

During the [play framework meetup in Mai 2015](http://www.meetup.com/Play-Berlin-Brandenburg/events/222130013/), there were 3 talks about Dependency Injection (DI) in Play 2.4:

- from Micheal: runtime DI in Java with [Guice](https://github.com/google/guice), the default framework introduced in play 2.4: [https://github.com/schleichardt/play-2-4-di-talk](https://github.com/schleichardt/play-2-4-di-talk)

- from [Oleg](https://twitter.com/easyangel): runtime DI in Scala with [Scaldi](http://scaldi.org/): [http://scaldi.github.io/scaldi-play-2.4.0-presentation/](http://scaldi.github.io/scaldi-play-2.4.0-presentation/)

- compile-time DI in Scala with constructor arguments or [MacWire](https://github.com/adamw/macwire): [http://de.slideshare.net/yann_s/play-24dimacwire](http://de.slideshare.net/yann_s/play-24dimacwire).
For that talk, I re-used the TBA application I used at the [Ping Conf](/blog/2014/02/17/ping-conf-2014/) last year and added a version using MacWire: https://github.com/yanns/TPA/tree/master/frontend/TBA_macwire
