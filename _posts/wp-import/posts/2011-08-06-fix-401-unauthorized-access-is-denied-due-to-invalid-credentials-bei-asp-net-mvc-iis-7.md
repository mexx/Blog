---
layout: post
title: "Fix: “401 - Unauthorized: Access is denied due to invalid credentials” bei ASP.NET MVC & IIS 7"
date: 2011-08-06 23:25
author: robert.muehsig
comments: true
categories: [Fix]
tags: [ASP.NET MVC, Fehler, Fix, HowTo, IIS7]
---
{% include JB/setup %}
<p>Diese Fehlermeldung kann dutzende Gründe haben, z.B. kann hat vielleicht der IIS Prozess keinen Zugriff auf die Dateien oder das Passwort des Accounts unter dem die Anwendung läuft ist falsch. Kurz um: Es ist eine nicht wirklich klar definierte Fehlermeldung. <a href="http://www.uniquesoftware.de/Blog/de/post/2011/07/22/Access-denied-due-to-invalid-credentials-aber-nur-manchmal.aspx">Martin</a> hatte bereits vor einiger Zeit von einem interessanten Problem geschrieben (bzw. hat er den Workaround <a href="http://blog.yeticode.co.uk/2011/03/iis7-iis-express-401-access-is-denied-due-to-invalid-credentials-issue/">hier</a> gefunden) . Die Symptome des Fehlers waren:</p> <p>- Fehlermeldung “401 - Unauthorized: Access is denied due to invalid credentials” von einer Remote-Maschine</p> <p>- In den IIS Logs steht nichts außergewöhnliches.</p> <p>- Die Seite wurde (via Remote Desktop/localhost) richtig angezeigt.</p> <p><a href="{{BASE_PATH}}/assets/wp-images/image1322.png"><img style="background-image: none; border-bottom: 0px; border-left: 0px; padding-left: 0px; padding-right: 0px; display: inline; float: left; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" align="left" src="{{BASE_PATH}}/assets/wp-images/image_thumb504.png" width="136" height="123"></a><strong>Nun zum eigentlich verwirrenden:</strong> Wenn man im IIS Manager (IIS Manager -&gt; Error Pages -&gt; Edit Feature Settings) auf Detailed Error stellt, geht die Seite plötzlich auch von aussen. Ohne Fehlermeldung. Mh?!</p> <p>Die Website war zudem eine ASP.NET MVC Seite, allerdings kann das Problem vermutlich auch beim normalen ASP.NET auftreten. Die Seite die wir eigentlich sehen wollten, war die Loginseite. Diese ging aber nicht.</p> <p><strong>Was haben die Detailed Error Einstellungen zu sagen?</strong></p> <p>Unter IIS Manager –&gt; Error Pages und Edit Feature Settings (auf der rechten Seite), kann man diese Einstellung vornehmen. Als Default ist folgendes gesetzt:</p> <p><a href="{{BASE_PATH}}/assets/wp-images/image1323.png"><img style="background-image: none; border-bottom: 0px; border-left: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="{{BASE_PATH}}/assets/wp-images/image_thumb505.png" width="402" height="344"></a></p> <p>Diese Einstellung besagt, dass die Detail-Informationen eines Fehlers dem Anwendung wenn er über localhost kommt angezeigt werden. Allen anderen wird eine “unspezifische” Seite gezeigt. Hier wird der Prozess <a href="http://learn.iis.net/page.aspx/267/how-to-use-http-detailed-errors-in-iis-70/">genauer</a> beschrieben. Wichtigste Stelle hier:</p> <p><a href="{{BASE_PATH}}/assets/wp-images/image1324.png"><img style="background-image: none; border-bottom: 0px; border-left: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="{{BASE_PATH}}/assets/wp-images/image_thumb506.png" width="544" height="281"></a></p> <p>Hier kommen wir zum Symptom “Die Seite wurde (via Remote Desktop/localhost) richtig angezeigt.”</p> <p>Was mir erst später auffiel war, dass die Seite zwar auf dem Server richtig angezeigt wurde, es aber mit Http Status 401 quittiert wurde. Der “Response Body” bestand jedoch aus der Loginseite, daher ist es erst nicht aufgefallen.</p> <p>Wie oben im Prozess abgebildet: Wenn der errorMode auf “detailed” steht, wird der Response Body der Fehlermeldung gerendert. In meinem Fall war der Response Body die Loginseite.</p> <p>Als wir es von dem Default Wert auf “Detailed Error” (also egal ob die Seite von localhost oder von Remote aufgerufen wurde) stellten, hatten wir dasselbe verhalten.</p> <p><strong>Warum war der Status Code 401?</strong></p> <p>Das Hauptproblem war: Ein “<strong>RenderAction</strong>()” auf der Layout/Masterpage, welche auf eine Action ging welche die Authentifizierung erforderte. Dies hat den 401 Fehler provoziert und als Ergebnis irgendwie die Loginseite in den Body getan. Auch wenn mir nicht ganz klar ist, warum der Fehler nicht am Ende in eine Art “Kreis” gekommen ist – weil die Loginseite auch die Layout benutzt. </p> <p><strong>TL;DR – die Problemlösung:</strong></p> <p>Prüft unbedingt ob nicht irgendwo ein <strong>RenderAction</strong> “<u>falsch” aufgerufen wird </u>wenn ihr diesen Fehler bei eurer Loginseite bekommt. </p> <p>Anderer Tipp noch: <u>Schaut via Firebug o.Ä. auf die</u> <strong>Statuscodes</strong> <u>der Seite</u>.</p> <p>Der Tipp kam übrigens von einer <a href="http://serverfault.com/questions/137073/401-unauthorized-on-server-2008-r2-iis-7-5">Antwort auf Stackoverflow</a> (die Frage hatte ein anderer gestellt, der die selbe Fehlermeldung + selben Symptome hatte)</p>
