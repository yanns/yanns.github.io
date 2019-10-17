+++
title = "DevOpsDays Berlin 2013"
date = 2013-05-08
path = "blog/2013/05/08/devopsdays-berlin-2013/"
+++

direct from DevOpsDay Berlin 2013, my notices:

<a target="_blank" href="http://www.devopsdays.org/events/2013-berlin/program/">Program</a>
<a target="_blank" href="http://new.livestream.com/accounts/4051563">Videos</a>

<h2 id="day1"><a href="#day1">Day #1</a></h2>

From the <a target="_blank" href="http://www.devopsdays.org/events/2013-berlin/proposals/DevOps3.0/">presentation from Immobilienscout</a> - <i>Marcel Wolf, <a target="_blank" href="https://twitter.com/felixsperling">Felix Sperling</a></i>
<ul>
	<li>Dev AM rotation: exchange between Dev and Ops</li>
	<li>Self service VM</li>
	<li>unique configuration server, accessible to anyone</li>
</ul>


<a target="_blank" href="http://www.devopsdays.org/events/2013-berlin/proposals/DevOps3.0/">DevTools team at Etsy</a> - <a target="_blank" href="https://twitter.com/mrtazz">Daniel Schauenberg</a>
<ul>
	<li><a target="_blank" href="http://fr.slideshare.net/mrtazz/devtools-at-etsy">Slides</a></li>
	<li><a target="_blank" href="https://github.com/etsy/deployinator">deployinator</a></li>
	<li>Each new employee should deploy on her first day.</li>
	<li><a target="_blank" href="http://lxc.sourceforge.net/">LXC container</a> for tests</li>
	<li>Statistics with <a target="_blank" href="https://github.com/etsy/statsd/">statsd</a>, <a target="_blank" href="https://github.com/etsy/logster">logster</a> and <a target="_blank" href="http://graphite.wikidot.com/">graphite</a></li>
	<li>Log streamer with <a target="_blank" href="https://github.com/etsy/supergrep">supergrep</a></li>
	<li><a target="_blank" href="https://github.com/etsy">Open Source projects</a></li>
</ul>


<h2 id="day2"><a href="#day2">Day #2</a></h2>

<a target="_blank" href="http://www.devopsdays.org/events/2013-berlin/proposals/How%20the%20QA%20team%20got%20Prezi%20ready%20for%20DevOps/">How the QA team got Prezi ready for DevOps</a> - <a target="_blank" href="https://twitter.com/pneumark">Peter Neumark</a>
<ul>
	<li><a target="_blank" href="http://prezi.com/qmrekeeqqvyf/how-the-qa-team-got-prezi-ready-for-devops/">presentation</a></li>
	<li>Like the <a target="_blank" href="https://twitter.com/simon_yann/status/339462592315682816/photo/1">error handling</a>: "When blame inevitably arises, the most senior people in the room should repeat this mantra: if a mistake happens, shame on us for making it so easy to make that mistake". Very similar to risk management culture.</li>
</ul>


<a target="_blank" href="http://www.devopsdays.org/events/2013-berlin/proposals/Podularity%20FTW/">Podularity FTW!</a> - <a target="_blank" href="https://twitter.com/tlossen">Tim Lossen</a>
<ul>
	<li><a target="_blank" href="http://fr.slideshare.net/tim.lossen.de/podularity-ftw">slides</a></li>
	<li>team = autonomous cell (even technological stack, product...)</li>
	<li>The organization is a supercell, bindings autonomous cells together.</li>
	<li><a target="_blank" href="https://twitter.com/simon_yann/status/339461227552047104/photo/1">Lunch roulette</a></li>
	<li>Interesting question from the audience about the business continuity: if each team can choose its technological stack, is not it a problem then the team change, and when the new members do not know the new stack he is working with? Tim answered that it was indeed a problem the organization though of, but in practice, it never happened.<br/>I like this approach that I could try to summarize like this: do not spend your time trying to avoid problems, but solve real problem that exist.</li>
</ul>



<a target="_blank" href="http://www.devopsdays.org/events/2013-berlin/proposals/How%20we%20built%20and%20deployed%20the%20Honshu%20way/">Island Life: How we built and deployed the Honshū way</a> - Wes Mason
<ul>
	<li>from one monolith app to several islands like components</li>
</ul>


<h2 id="other-notices"><a href="#other-notices">Other notices:</a></h2>
<ul>
	<li><a target="_blank" href="http://hubot.github.com/">robot for chat room</a> to post more information</li>
	<li>secrets are hard to deploy in a secure way</li>
	<li>distributed file system: <a target="_blank" href="http://ceph.com/">ceph</a><br/>more info: <a target="_blank" href="http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.110.4574&rep=rep1&type=pdf">Ceph: A scalable, high-performance distributed file system (2006)</a>
</li>
	<li><a target="_blank" href="http://www.docker.io/">doker.io</a> to manage linux containers. The demo was impressive, deploying one version and then another one in a few minutes.</li>
	<li>Discussion with <a target="_blank" href="http://www.gutefrage.net/">gutefrage.net</a> who use Scala / <a target="_blank" href="https://github.com/twitter/finagle">Finagle</a> with <a target="_blank" href="http://thrift.apache.org/">Thrift</a>.<br>Storage of statistics with <a target="_blank" href="http://opentsdb.net/">OpenTSDB</a></li>
	<li><a target="_blank" href="http://zeroturnaround.com/software/liverebel/">LiveRebel</a>: ZeroTurnaround made a presentation of LiveRebel<br/>
LiveRebel contains some versions of the application.<br/>
These versions can be uploaded, manually or automatically (maven plugin, command line tool...)<br/><br/>
Then LiveRebel can deploy a specific version on production (or staging...)<br/>
For this, an agent is running on each server.<br/><br/>
LiveRebel can deploy to one server, check it with some configured smoke test.<br/>
If the test is successful, the server is activated on the cluster, and the deployment process continues with the next server.<br/><br/>
Liquibase is used to deploy a version to a database.</li>
</ul>
