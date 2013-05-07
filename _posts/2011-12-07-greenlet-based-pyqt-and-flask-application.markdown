---
date: 2011-12-07 19:02:18
layout: post
title: Code <small>Greenlet based PyQt and Flask Application</small>
tags:
- greenlet
- PyQt
- Flask
- python
---

I had a slightly odd moment a few weeks ago and felt that I really didn't want to write a PyQt based desktop application.
Instead, fueled by some webapp based docs i'd been reading recently (I blame you @manneohrstrom) which were written using Flask. I decided to try out launching a PyQt QWebView and pointing it back at a local webservice run from the same process.

What I great idea I thought! Jumping in blindly only to quickly discover that both Flask and PyQt want to run in the primary python thread. To get around the problem I fell back onto Greenlets and hand processing events in PyQt rather than giving over control to app.\_exec\(\).

Ultimately I ended up ditching the whole thing and reverting back to a vanilla
PyQt application since once I started to add user interaction for elements such
as dynamic combo boxes I found that my Javascript skills were sufficiently rusty
that it was taking an age to write the handlers, and callbacks on both ends of
the system.

{% highlight python %}
#!/usr/bin/env python

import sys
import time

from flask import Flask
from PyQt4 import QtCore, QtGui, QtWebKit

import gevent.wsgi

class WebView(QtWebKit.QWebView):
    def __init__(self):
        QtWebKit.QWebView.__init__(self)

        self.load(QtCore.QUrl("http://localhost:5000/"))
        self.connect(self , QtCore.SIGNAL("clicked()"), self.closeEvent)

    def closeEvent(self, event):
        self.deleteLater()
        app.quit()
        print "closing gui"
        g.kill(gevent.GreenletExit, block=False)
        f.kill(gevent.GreenletExit, block=False)
        print "are gui and webserver alive? ", g.dead, f.dead


class PyQtGreenlet(gevent.Greenlet):
    def __init__(self, app):
        gevent.Greenlet.__init__(self)
        self.app = app

    def _run(self):
        while True:
            self.app.processEvents()
            while self.app.hasPendingEvents():
                self.app.processEvents()
                gevent.sleep(0.01)
        gevent.sleep(0.1)

if __name__ == "__main__":
    fapp = Flask(__name__)
    fapp.debug=True

    @fapp.route("/")
    def hello():
        print time.time()
        return '<a href="http://localhost:5000/bye" > quit() </a></br><a href="http://localhost:5000/" > reload </a></br>count={0}'.format(time.time())

    @fapp.route("/bye", methods=['GET'])
    def goodbye():
        print "killing webserver"
        f.kill(block=False)
        print "is webserver alive? ", f.dead
        window.closeEvent(QtCore.SIGNAL("clicked()"))
        return 'goodbye'

    http_server = gevent.wsgi.WSGIServer(('', 5000), fapp)
    f = gevent.spawn( http_server.serve_forever)

    app = QtGui.QApplication(sys.argv)
    window = WebView()
    window.show()
    g = PyQtGreenlet.spawn(app)
    gevent.joinall([f, g])
{% endhighlight %}
