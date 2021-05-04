---
title: "Listen to Youtube Livestreams From the Terminal"
date: 2020-06-06
tags: ["terminal", "music", "mpv", "streaming", "streamlink", "go", "sockets", "unix"]
categories: ["cool-stuff"]
---

Like many people, while working I like to have some music on in the background. Normally this is either a Spotify playlist of my own, or some Youtube livestream of jazz, lofi or another genre that is mostly instrumental.

Also, I spend a lot of time in the terminal while working. Putting two and two together I thought it would be cool to be able to have a CLI to play the audio from different livestreams.

The basic features I'd like are:
- Play the audio of a Youtube livestream, given the URL
- Pause the current livestream
- Save the url of this livestream as a favourite
- Alter the volume of playback

A quick search and I found [streamlink](https://streamlink.github.io). Soon after, I got feature number one working by simply using streamlink and running:

```bash
streamlink --player "/usr/bin/mpv --no-video" https://www.youtube.com/watch?v=8lPn3PZkA_M best
```

This will launch the player I have installed on my machine (mpv) without showing the video, of the live stream at the given URL with the best quality possible.

This is all fine and dandy except, as far as I am aware, you cannot then interact with the stream any further (because it's in no video mode). So, we need a way of sending commands to and from mpv once it has started playing the stream.

Luckily mpv has a great [manual](https://mpv.io/manual/master) that explains how to use the CLI.

I noticed the JSON IPC section which enables you to send and receive commands through a unix socket or a named pipe using the `--input-ipc-server` option.

On that topic, I recently came across sockets while reading [Black Hat Go](https://nostarch.com/blackhatgo) and so it seems fitting to writing a little script to send commands to the input server in Go.

*I should note here that my aim is to actually write a Vim plugin after this (something I've not done before) so that I can actually control this tool inside of Vim, which is where I do all of my programming. So stay tuned for the next post.*

Now it's time for some planning...

The Go binary should simply provide a wrapper around the commands so that we don't have to do any weird JSON formatting and piping with `socat` ourselves. This means that this:
```bash
echo '{ "command": ["set_property", "pause", true] }' | socat - /tmp/mpvsocket
```
turns to this:
```bash
sycli pause
```

Go makes this incredibly easy, as we will see. All we will need to do is parse the command line arguments and then generate the relevant structs, encode them and send them to the socket. We can then wait for a response and print any errors when they inevitably come up.

### Creating the structs
We want to model the JSON using structs. The JSON model is actually very simple for interacting and receiving events from the socket:

**Sending**
```json
{ "command": ["command_name", "param1", "param2", ...] }
```

**Receiving**
```json
{ "error": "success", "data": null }
```
You can also receive events from the socket but for this use case I'm not interested.

From this let's create our models:
```go
type SendWrapper struct {
	Command   Command `json:"command"`
	RequestID int     `json:"request_id,omitempty"`
	Async     bool    `json:"async,omitempty"`
}

type ReceiveWrapper struct {
	Error     string      `json:"error"`
	RequestID int         `json:"request_id"`
	Data      interface{} `json:"data"`
}

type Command struct {
	CommandName string
	Params      []interface{}
}
```
The content inside the backticks allows you to specify names of the fields in JSON and also whether to include the field or not if it is empty. 

Maybe you've noticed but our `Command` struct isn't actually in the same format as we need to send it. I've written it like this because I thought it made more sense but we could have gone either way. It also gives me the opportunity to show you how to marshal your own types into custom JSON!

```go
func (c Command) MarshalJSON() ([]byte, error) {
	all := append([]interface{}{c.CommandName}, c.Params...)
	return json.Marshal(all)
}
```
It is actually very simple. We write a method on our `Command` struct called `MarshalJSON`. This is only a small one but you could generate any sort of json string you like here. I'm just appending the command name to the front of the parameters slice and then having the json package encode that for me.

Sweet, now we have modelled our objects for sending and receiving commands. Next up is how to connect and communicate with the mpv socket!

We will be using Go's `net` package here:

```go
func connect() net.Conn {
	conn, err := net.Dial("unix", "/tmp/mpvsocket")
	if err != nil {
		log.Fatalln(err)
	}
	return conn
}
```

Simple! Dial a connection on the unix network to the socket that we want.

Now we have a connection, let's write the function to send our Command structs to mpv.

```go
func SendCommand(cmd Command, conn net.Conn) (int, error) {
	wrapper := SendWrapper{Command: cmd}
	b, err := json.Marshal(wrapper)
	if err != nil {
		return 0, err
	}
	return conn.Write([]byte(string(b) + "\n"))
}
```
First we wrap our command in the send wrapper, at this stage I'm not interested in the async or request_id values. When we call `json.Marshal` it will subsequently call our `MarshalJSON` method when it comes to encode the Command field and thus our correclty formatted array of arguments will be generated as the command field of the JSON object. Following from that, we use our open connection to the Unix socket to write the bytes of the JSON string. Notice I've had to include a newline on the end as the IPC server for mpv requires all commands to end with a newline.

So a quick example of how we would use this code to pause the currently running mpv player is as follows:
```go
func pause() {
	conn := connect()
	defer conn.Close()
	com := sycli.SetBoolPropertyCommand("pause", true)
	if _, err := sycli.SendCommand(com, conn); err != nil {
		log.Fatal(err)
	}
}
```
I've written a little utility command called `SetBoolPropertyCommand` which just constructs the Command struct for you in the form of an option that takes a single argument of a bool.

So that is a quick overview of how I made a little CLI in Go to interact with mpv, allowing me to control the player in order to listen to Youtube livestreams from the command line!

I'll probably add a few more things to the project in the future such as more commands, but it's not intended to be anything sound and polished - just a fun little hacky project. Although, as previously mentioned I do want to write a Vim plugin to wrap around this so I can interact with it from Vim.

Please take a look at the full code in [my repo](https://github.com/shnupta/sycli).

--------------
This is the second post of my [weekly post challenge!]({{< ref "weekly-post-challenge.md" >}})
