---
layout: post
title: "Handling data streams with Play2 and Server-Send Events"
date: 2012-08-12 09:42
comments: true
categories:
- Play Framework
- HTML5
---
<h2 id="Handling-data-streams"><a href="#Handling-data-streams">Handling data streams</a></h2>
As the version 2 of Play! Framework was published, I was very interested in its new capabilities to handle data streams reactively.<br/>
As a technical proof of concept, I wrote a parser that works with chunks of data instead of loading the whole content in memory.
My source was a file containing the geographical coordinates of Wikipedia articles.<br/>
(This file is the result of an experience of Triposo, showing how Wikipedia has spread over the planet since the start of the Wikipedia project. Do not forget to watch the other labs from Triposo, they are great!)<br/>

Play2 architecture is based on event, and gives us some tools to work with streams of data:
<ul>
	<li><a target="_blank" href="http://www.playframework.com/documentation/2.2.x/Enumerators">Enumerators</a> produce chunk of data</li>
	<li><a target="_blank" href="http://www.playframework.com/documentation/2.2.x/Enumeratees">Enumeratees</a> transform these chunks</li>
	<li><a target="_blank" href="http://www.playframework.com/documentation/2.2.x/Iteratees">Iteratees</a> consumes these chunks</li>
</ul>

(For more information, you can read:
<ul>
	<li><a target="_blank" href="https://gist.github.com/sadache/3072893">Is socket.push(bytes) all you need to program Realtime Web apps?</a> from <a href="https://twitter.com/Sadache">Sadek Drobi, CTO Zenexity</a>.</li>
	<li><a target="_blank" href="http://greweb.me/2012/08/zound-a-playframework-2-audio-streaming-experiment-using-iteratees/">Zound, a PlayFramework 2 audio streaming experiment using Iteratees</a> from <a target="_blank" href="https://twitter.com/greweb">Gaetan Renaudeau</a></li>
	<li>If you understand french, you can read <a target="_blank" href="http://www.touilleur-express.fr/2012/08/05/realtime-web-application-un-exemple-avec-play2/">Realtime Web Application, un exemple avec Play2</a> from <a target="_blank" href="https://twitter.com/nmartignole">Nicolas Martignole</a>)
</li>
</ul>




The production of data is an Enumerator, sending line after line of the input file:
```scala
def lineEnumerator(source: Source) : Enumerator[String] = {
  val lines = source.getLines()

  Enumerator.fromCallback1[String] ( _ => {
    val line = if (lines.hasNext) {
      Some(lines.next())
    } else {
      None
    }
    Future.successful(line)
  }, source.close)
}
```
With an Enumeratee, each line can be possibly parsed into a Coordinate class:
```scala
val lineParser: Enumeratee[String, Option[Coordinate]] = Enumeratee.map[String] { line =>
  line.split("\t") match {
    case Array(_, IsDouble(latitude), IsDouble(longitude)) => Some(Coordinate(latitude, longitude))
    case _ => None
  }
}
```
The Enumerator can be composed with an Enumeratee with:
```scala
val source = scala.io.Source.fromFile(Play.getExistingFile("conf/coosbyid.txt").get)
lineEnumerator(source) &> lineParser
```

I know, the method's name "&>" can make some of you go away. Please stay! This sign is like the pipe in bash. It is very easy to understand:
```scala
lineEnumerator(source) &> lineParser
```
is the same as (removing infix notation)
```scala
lineEnumerator(source).&>(lineParser)
```
which is the same as (method alias)
```scala
lineEnumerator(source).through(lineParser)
```
Use the last form if the first one is not your taste... :)

With a last Enumeratee to produce JSON, I can send the stream directly to the browser with Server Send Events.
```scala
val source = scala.io.Source.fromFile(Play.getExistingFile("conf/coosbyid.txt").get)
val jsonStream = lineEnumerator(source) &> lineParser &> validCoordinate &> asJson
val eventDataStream = jsonStream &> EventSource()
Ok.chunked(eventDataStream).as("text/event-stream")
```

What to notice:
Only chunks of data are in memory. The whole content of the source file is never loaded completely.
Each step of the process is isolated in an Enumertor or Enumeratee, making it very easy to modify, to re-use, to combine in a different way.
The Enumerator is reading a file, but you can imagine it could read data from a web service, of from a database.


<h2 id="SSE"><a href="#SSE">Server-Send Events</a></h2>
When we want to send events in "real time" to the browser, what technologies are available?
<ul>
	<li>polling: the browser pools the server every x milliseconds to check if there is a new message. This method is not very efficient, because a lot of requests are necessary to give the illusion to update the application in real time.</li>
	<li>long-polling (or <a target="_blank" href="http://en.wikipedia.org/wiki/Comet_(programming)">Comet</a>): the browser opens a connection to the server (for example in a iframe), and the server keeps the connection opened. When the server wants to push data to the client, it sends this data with the opened connection. The client receives the data, and opens a connection again for further messages. With this method, the browser is always showing that it is waiting for data. This technology does not scale on threaded system, as each opened connection uses a thread. In JEE environment, we need an asynchronous servlet 3.1 not to make the server exploding.</li>
	<li><a target="_blank" href="http://dev.w3.org/html5/eventsource/">Server-Send Events (SSE)</a> are quite similar to Comet. The main difference is that the browser manages this connection. For example, it opens the connection again if it falls.</li>
	<li><a target="_blank" href="http://dev.w3.org/html5/websockets/">WebSockets</a> provide a bi-directional, full-duplex communications channels. It is a different protocol than HTTP.</li>
</ul>



I choose to use Server-Send Events instead of WebSockets because of the following reasons:
<ul>
	<li>I've already played with WebSockets and wanted to try something new.</li>
	<li>WebSockets are great and can communicate in both directions. But this technology is a new protocol, sometimes difficult to integrate in an existing infrastructure (Proxy, Load-Balancer, Firewall...) Server-Send Events, on the other hand, use the HTTP protocol. The PaaS <a target="_blank" href="https://www.heroku.com/">Heroku</a> does not support WebSockets yet, but support SSE. When pushing data from the server to clients is all what you need, SSE can be what is the most appropriate and is <a target="_blank" href="http://caniuse.com/#search=eventsource">well supported</a> (except in IE for the moment)</li>
</ul>



<h2 id="SSE-API"><a href="#SSE-API">Server-Send Events API</a></h2>
The Javascript API is very simple:
```javascript
feed = new EventSource('/stream');

// receive message
feed.addEventListener('message', function(e) {
    var data = JSON.parse(e.data);
    // do something with data
}, false);
```

<h2 id="Visualizing-results"><a href="#Visualizing-results">Visualizing the results</a></h2>
As the stream is sending coordinates, my first attempt was to display them on a earth in 3D. For this, I used three.js, which was very simple. The first results were promising, but sadly, the browser could not display so much information in 3D. I had to found an alternative.<br/>
My second attempt was to display these coordinates on a 2D canvas, and that worked well, although less impressive that a 3D map... :)<br/>
You can see the result on Heroku: http://wiki-growth.herokuapp.com/<br/>
The code source is available on github: https://github.com/yanns/play2-wiki-growth-sse<br/>
You can run it by yourself with Play and let heroku sleeping.