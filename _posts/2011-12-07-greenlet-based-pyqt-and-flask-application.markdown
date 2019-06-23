---
date: 2011-12-07 19:02:18
layout: post
current: post
title: Greenlet based PyQt and Flask Application</small>
tags:
- Python
- Qt
- Flask
class: post-template
subclass: post
author: andrew
navigation: true
---

This is an old trick which could be used in the past to allow a webapp to be
written and displayed on the desktop without the need to launch a full web
browser.

Here we launch a PyQt QWebView and redirect it back at a local web service
run in the same process.

The trick is to get around the problem of the web framework and Qt both wanting
control of the primary python thread to execute their control loops.  To get
around the problem each can be spawned into their own Greenlet and then forcing
a manual execution of Qt events rather than giving over control to `app._exec()`.

{% highlight python %}
#!/usr/bin/env python

import logging
import sys
import time

from flask import Flask
import gevent.wsgi
from PyQt4 import QtCore, QtGui, QtWebKit


_LOGGER = logging.getLogger("QtFlaskApp")


class WebView(QtWebKit.QWebView):
    def __init__(self):
        QtWebKit.QWebView.__init__(self)

        self.load(QtCore.QUrl("http://localhost:5000/"))
        self.connect(self , QtCore.SIGNAL("clicked()"), self.closeEvent)

    def closeEvent(self, event):
        self.deleteLater()
        app.quit()

        _LOGGER.info("closing gui")

        g.kill(gevent.GreenletExit, block=False)
        f.kill(gevent.GreenletExit, block=False)

        _LOGGER.debug("are gui and webserver alive? %s %s", g.dead, f.dead)


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
    logging.basicConfig()  # replace with more controlled logging in a real app.

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

