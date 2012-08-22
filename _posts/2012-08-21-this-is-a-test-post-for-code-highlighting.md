---
layout: post
title: "This is a test post for code highlighting"
description: ""
category: 
tags: []
---
{% include JB/setup %}
JavaScript
---
	var http = require('http');
	var fs = require('fs');
	var index = fs.readFileSync('index.html');

	http.createServer(function (req, res) {
	  res.writeHead(200, {'Content-Type': 'text/plain'});
	  res.end(index);
	}).listen(31337);

Ruby
---
	def getCostAndMpg
	    cost = 30000  # some fancy db calls go here
	    mpg = 30
	    return cost,mpg
	end
	AltimaCost, AltimaMpg = getCostAndMpg
	puts "AltimaCost = #{AltimaCost}, AltimaMpg = #{AltimaMpg}"

C#
---
	public class Chat : Hub
	{
	    public void Send(string message)
	    {
	        // Call the addMessage method on all clients
	        Clients.addNotification(message);
	    }
	}
