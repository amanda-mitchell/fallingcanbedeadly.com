---
title: MonkeySpace Talk Slides and Notes
layout: post
---

I delivered my first conference talk at MonkeySpace yesterday. The video will be available at some point, but in the meantime, here are the slides and some notes to help make sense of them.

<script async="true" class="speakerdeck-embed" data-id="e4d27400d88d0130de013addc031b9b8" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js">
</script>
<noscript>If you had Javascript enabled, you'd see a <a href='https://speakerdeck.com/davidmitchell/server-side-mono-how-ready-is-it'>slide deck</a> here.</noscript>

### Overview

Mono is a potentially attractive option for organizations that are already using ASP.NET because IIS is a pain to configure and Windows licenses can get really expensive.

Many libraries indicate that they support Mono, but there is little information about using it for web sites and services beyond basic, "getting started" tutorials. I'm trying to document and publish my experiments as I go along to help others who might be trying this and in the hope that more people will start documenting their experiences, too.

The major snags I encountered were:
* xbuild doesn't have a good fallback mechanism for unkown profiles (we use the Client Profile for some assemblies)
* Microsoft's format for Forms Authentication cookies is undocumented and difficult to reverse engineer. (see my [earlier post](http://www.fallingcanbedeadly.com/posts/decoding-forms-authentication-cookies-with-mono) on this topic)
* Mono's support for SQL Server hasn't received significant updates for 2008 and 2012.
* [New Relic](http://newrelic.com)—an awesome monitoring service we use—doesn't support Mono.

I worked around all of these issues and even built my own version of the New Relic Profiler (linked below). In testing a simple service, I encountered very inconsistent performance that was quite sensitive to GC pauses.

There was a period of about seven minutes during which the performance of the Mono version of the service was as good as—or better than—its Windows counterpart, so I plan to continue investigating. These are some initial areas I intend to look into:

* Is the New Relic profiler itself causing noise?
* Does Mono 3.2 solve this issue? (I tested with one of the 3.0.x releases)
* Is there something about the application's architecture that causes difficulty for sgen?
* Would a stack other than ASP.NET MVC give me better throughput?

### Image Credits

Several of the images that I used were licensed under Creative Commons. Here are links to their original locations:

* ["Why?" by Kevin Lau](http://www.flickr.com/photos/87473264@N00/348551209/in/photolist-wNq8B-BsSBE-BsSCP-CC26F-DmTxu-Fh7Tf-FrYhM-KS9kf-Lw1Ar-LPdRU-MXN5u-NPVmd-NTVYC-24YVt1-2CRkgx-348sMH-38Vb5G-3ijD4C-3nLMEw-3N9cQu-48nN8D-4nzuyL-4nEpqG-4oU4d1-4rhuDV-4yrA7c-4BrzRm-4EyxPB-4G38Lv-4Hg2Qx-4HS4Jp-4HXvXF-4QDR2i-4UnLpK-4VT7yL-553Xsd-55R1Tu-56zThs-5959u3-5aghr9-5crX6o-5dsRpT-5fibHe-5iNqEp-5ng2kr-5o5Ga8-5oKXcg-5q3WYL-5sTkGu-5CgFn1-5FLBDT)
* ["Forex Money for Exchange in Currency Bank" by epSos .de](http://www.flickr.com/photos/36495803@N05/8463683689/in/photolist-dTUAhR-dUSc9a-9dGPED-agvquQ-b7NFzr-dmyfCP-9ZA9J6-aYWk56-aFAaK6-aFATbM-dSZe91-bt4mNt-bH1iX8-chEwR9-cnchKE-aFDjPB-8usD9K-bzmxxS-aFAQEv-bta55K-bZvUDS-brd1K2-aFAPtx-bi1bhM-a2YSa6-a2Y7cz-dB7F8e-889HV4-bta3kH-9q3RuT-aWUuqV-ayZVrf-biaRHp-89qVPo-8GE2S4-buW1s2-9t6RfD-8HWvej-aarLAr-9kJxyv-8ipnFE-9wUujx-7HKuTk-9NNJjU-egE3Qz-7B552e-7M4MWJ-a9JfVU-dLLgir)
* ["History in 12 Minifigs" by R D L](http://www.flickr.com/photos/7269902@N07/7136403625/)
* ["Lots of cats" by star5112](http://www.flickr.com/photos/johnjoh/321893075/in/photolist-urMAB-vVqN6-Ai5pW-FwjdL-GMKdR-JKHDS-Mxen8-NKvmt-2ahVgP-31ovCr-31ovFZ-37grTZ-3e2NRt-3iEAoL-4bbvhk-4c9R3V-4du4o4-4dFzoR-4iuDMX-4wKxDh-4CGAqF-4FXeKn-4JL9ey-4WpFx7-564PmD-57hZaz-57i13P-57na3u-57nc6Y-5ajoq4-5pKB3t-5rTHim-5t6zg6-5t6zPz-5t6Aw2-5vJJSz-5vZveg-5wUa9b-5GKAEG-5GN89s-5HtB23-5NyQjC-5RUp6K-5SEzEa-5UzE2k-5ZKpW3-5ZKqnu-619XaV-6cWRC7-6gFSf5-6i5VLY/)
* ["Whack A Mole Fever" by TPapi](http://www.flickr.com/photos/11483960@N08/2765541278/in/photolist-5do85b-5iUes7-5mnk9i-5mrApb-5o6Ekm-5uLcFr-5wf4nR-5CbAnt-5PXSRw-5RC6W9-5T46tX-5Vm5ou-6darm5-6jfnWP-6wJT7C-6xmosH-6ChpLF-6GkyZe-6LLmHD-6M1SiA-6NqF1j-6Px47W-6SfAhg-6TPqGP-6TPqPH-6XHi6q-6Zqdxz-75myaX-76XxoE-79zCDj-79ACow-7r9TAS-ankbfD-8yAS1d-8KNvPy-8yARHo-bhWimK-7Zafgf-aF6Qco-8SJLpt-8dQjyK-bLuHoD-dudGyw-85kx1H-cXY2dw-ce2jXL-a1zjeT-8Hupa5-9EWf6o-amruxf-85oFoU)
* ["Basement Cat wuz once in Ceiling" by Noah Friedman](http://www.flickr.com/photos/65267324@N00/2634354055/in/photolist-51MKFv-57jmsB-5jc7yc-5kwQKi-5oJjzt-5ugwTs-5A6bTc-5BGBtF-5Cd2Si-5Cz2bY-5DPEGh-5GZpQ3-5HfaBT-5JmWkJ-5KufLN-5NfPwD-5WmFHh-5ZtjUM-61rX5o-625ALm-64KSFx-655rbv-659Hio-659HWQ-65JeoN-67p7kx-67p7q4-68pTzx-68qxWn-68qK4c-68uKSd-68uXoG-68uXrj-68uXtL-68uXB3-692ema-6aJP8t-6bn3P6-6cX1di-6cX1Hp-6dhGT7-6dLrMh-6e15mK-6fCUkN-6j9TM8-6j9Vte-6j9YAT-6ja6Jz-6jamxR-6jaqFv-6jat7H)
* ["Cat Attack - Computer Scenario 56/365 V" by static416](http://www.flickr.com/photos/ehacke/4584260984/)
* ["Cobweb abandoned, like the building" by simpson6642](http://www.flickr.com/photos/simpson6642/6040658117/in/photolist-acMXcT-aZepzB-cLR3N1-akHNg8-cioUk3-93janC-98AXzu-8bdgCv-dt1omG-8PYk5e-dFz6YJ-aknMcM-8uqsJk-7XLFJ8-f76baM-7zD1YF-8FnFyd-8qWpC4-dGt1po-eULJqW-akrpug-8cz7sx-8Eywu1-8FE5eD-aQWB3t-8F7LJJ/)
* ["Binoculars portrait (dscn4659_mod_vign_sm)" by gerlos](http://www.flickr.com/photos/gerlos/3119891607/)
* ["A herd of yaks passing the campsite" by Gavin Bishop](http://www.flickr.com/photos/a_brit_abroad/7932306622)
* ["P365 alternate image" by Katie](http://www.flickr.com/photos/12488254@N06/4307204153/in/photolist-7yBxJz-9eUjAs-9eUjLy-9eUm4U-9eRctM-a78nij-8TQzL5-8Hwp6J-9eUn6N-9eUni3-9eUmUb-9eUmEf-9eRdYk-9trQRJ-9bk8tQ-cCsr3o-bDYRzP-9HvYkQ-9Htdq6-9eUkbj-9eRdbZ-9eRcS2-9rSRkX-bHwjfK-dpvNPg-bHv7Jv-e4Cjo1-cRecju-aAZD4T-bgKm64-ck98oC-951GGT-9662C7)
* ["alone" by Chad Nicholson](http://www.flickr.com/photos/icopythat/6500632/)

### Other Links

* Projects advertising support for Mono
	* [ServiceStack](http://servicestack.net/)
	* [Nancy](http://github.com/NancyFx/Nancy)
	* [SignalR](http://github.com/SignalR/SignalR)
* Sites known to be using Mono
	* [Xamarin](http://xamarin.com) (the HTTP headers say they're using nginx and ASP.NET MVC 3)
	* [ServiceStack](https://twitter.com/demisbellot/status/349585542641487872) (problems with the setup are indicated [here](https://twitter.com/demisbellot/status/349586375428931586))
* Documentation for setup
	* [Forms Authentication on Mono](http://www.mono-project.com/Config_system_web_authentication)
	* [Getting HTTPS to work on Mono](http://www.mono-project.com/FAQ_Security)
* Code that I wrote while researching this topic
	* [How to parse a .NET Forms Authentication Cookie](https://gist.github.com/david-mitchell/5350992)
	* [Shim between the .NET Profiling API and the New Relic Profiler](https://github.com/david-mitchell/NewRelicShim)
	* ["Rough Draft" of method rewriting API in Mono](https://github.com/LogosBible/mono/tree/methodrewrite)
	* [New Relic profiler for Mono](https://github.com/david-mitchell/NewRelicProfiler)
* Other
	* [Joke about yak-shaving on twitter](https://twitter.com/carlfish/status/357289566866120705)
	* [New Relic's lack of support for Mono](https://newrelic.com/docs/dotnet/new-relic-for-net)
	* [Limitations of the .NET Profiling API](http://msdn.microsoft.com/en-us/library/bb384493.aspx)
