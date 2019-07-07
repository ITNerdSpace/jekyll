---
layout: post
published: true
author: Alexandre Dumont
last_modified_at: '2019-07-07 13:05 +0200'
title: Control an HS100 switch from Node-Red via TP-Link Cloud API
image: /images/tp-link_hs100_nodered_helloWorld.png
---
In this post I'll show how to create a Node-Red subflow to control a TP-Link switch (power on/power off), using the TP-Link Cloud API.

First, we'll create a new flow that looks like this:

![nodered-flow-tplink-hs100.png]({{site.baseurl}}/images/nodered-flow-tplink-hs100.png)

To help with the process, you just have to create a new flow in Node-Red, go to this [Gist](https://gist.github.com/adumont/dbc679320ca68f6c1a149c2f4fe9a439) and import the flow from clipboard (copy the following block, and in Node-Red go to the Menu, Import, Clipboard).

Double click on the **login payload** node to edit its code, and replace the USERNAME and PASSWORD strings with your own TP-Link credentials, as shown below:

![edit_credentials.png]({{site.baseurl}}/images/edit_credentials.png)

Now select all the nodes on the flow, and go to Node-Red menu, Subflow, Selection to subflow. That will create a subflow with all the nodes and place it on the palette as a new custom node.

Once in the subflow, we'll add 1 input and 1 output nodes (from the toolbar):

![subflow-properties.png]({{site.baseurl}}/images/subflow-properties.png)

We'll link the input to RBE node and the output to set_relay_state node, as shown:

![add-input-output-to-subflow.png]({{site.baseurl}}/images/add-input-output-to-subflow.png)

We can also rename the subflow. In my case I called it HS100. Notice how it appears on the palette as a new node type:

![subflows_in_the_palette.png]({{site.baseurl}}/images/subflows_in_the_palette.png)

Now we can add this new HS100 node to any Flow in our Node-Red. For example, I made this quick HS100 Hello World flow:

![tp-link_hs100_nodered_helloWorld.png]({{site.baseurl}}/images/tp-link_hs100_nodered_helloWorld.png)

Now you can easily do all kind of great stuff with your TP-Link Cloud devices, and control them from MQTT, a POST payload, a Telegram bot or even Google Assistant...



