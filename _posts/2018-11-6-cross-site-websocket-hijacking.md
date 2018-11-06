---
layout: post
title:  "Cross-Site Websocket Hijacking"
date:   2018-11-6 17:34:11 +0200
categories: [tech, rails, websockets, security]
---

Hi there! My name is Tony and I have been building an authentication system for websockets recently. There is a main pitfall called cross-site websocket hijacking (CSWSH). I will tell you about it in this article.

WebSockets are widely used in web apps nowadays and frequently are vulnerable to cross-site hijacking. I am going to show you how to reveal the vulnerability and also how to prevent this attack vector.

## Basics

WebSocket is a computer communications protocol, providing full-duplex communication channels over a single TCP connection.

The WebSocket API is an advanced technology that makes it possible to open a two-way interactive communication session between the user's browser and a server. With this API, you can send messages to a server and receive event-driven responses without having to poll the server for a reply.

## Rails & ActionCable

A commonly used pattern to authenticate a user is by using cookies:

{% highlight ruby %}
# app/channels/application_cable/connection.rb
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user

    def connect
      self.current_user = find_verified_user
    end

    private
      def find_verified_user
        if verified_user = User.find_by(id: cookies.encrypted[:user_id])
          verified_user
        else
          reject_unauthorized_connection
        end
      end
  end
end
{% endhighlight %}

Even though this approach with the encrypted cookies seems secure enough it is vulnerable to CSWSH. An example is taken from the official Ruby on Rails guides.

The reason is that websockets are not restrained by the same-origin policy. So, a websocket handshake request can be easily initiated from a malicious webpage. Due to the fact that this request is a regular HTTP(S) request, browsers happily send the cookies along, even cross-site.

## Reveal

It is easy. Just try to connect from a different host. This snippet can be used:

{% highlight javascript %}

var wsUri = "ws://localhost:3334/cable";

var output;

function init()
{
  output = document.getElementById("output");
  testWebSocket();
}

function testWebSocket()
{
  websocket = new WebSocket(wsUri);
  websocket.onopen = function(evt) { onOpen(evt) };
  websocket.onclose = function(evt) { onClose(evt) };
  websocket.onmessage = function(evt) { onMessage(evt) };
  websocket.onerror = function(evt) { onError(evt) };
}

function onOpen(evt)
{
  writeToScreen("CONNECTED");
  var obj = {"command":"subscribe","identifier":"{\"channel\":\"AccountChannel\"}"};
  var myJSON = JSON.stringify(obj);
  doSend(myJSON);
}

function onClose(evt)
{
  writeToScreen("DISCONNECTED");
}

function onMessage(evt)
{
  writeToScreen('<span style="color: blue;">RESPONSE: ' + evt.data+'</span>');
}

function onError(evt)
{
  writeToScreen('<span style="color: red;">ERROR:</span> ' + evt.data);
}

function doSend(message)
{
  writeToScreen("SENT: " + message);
  websocket.send(message);
}

function writeToScreen(message)
{
  var pre = document.createElement("p");
  pre.style.wordWrap = "break-word";
  pre.innerHTML = message;
  output.appendChild(pre);
}

window.addEventListener("load", init, false);
{% endhighlight %}

Although Burp Suite or similar can be used to verify this vulnerability by changing Origin header I recommend to open a straightforward connection and check that messages can be actually received (not only the handshake is succeeded), because the developer could have placed the Origin verification logic along with the access control checks.

## Mitigation

1. As has already been mentioned, verify Origin header of the handshake request
2. Or switch from cookies to localStorage as Meteor does

