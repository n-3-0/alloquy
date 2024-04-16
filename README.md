# Alloquy

This repo will be filled in with more detail later, but here is the tl;dr:

- Open source discord clone
- Stitches together existing infrastructure with a middle layer to unify everything
- Federate user accounts (explicitly opt-in), but separate everything else

## Game Plan

In order to make Discord, there are several key features that need to work:

- Users can create and manage accounts
- Rich text message channels
- Direct messages between users
- Group DMs between one or more users
- Voice channels
- Webcams in voice channels
- Screen sharing in voice channels
- Attachments/embeds in messages
- Voice and video in group/direct chats
- Server management

To build it out, I'm going to leverage the following architecture:

### IRCv3 via inspIRCd

IRC will handle basic messaging features, from channels to direct messages. All communication will be encrypted via TLS _at minimum_, and stored messages can be encrypted-at-rest through some means I'll figure out later. The most important part is IRCv3, which enables critical features necessary for this system:

- Strict Transport Security
    - Ensures downgrade attacks are mitigated when using external clients
- Capability Negotiation
    - Enables upgrading security for DMs from standard to E2E encryption
- Message Tags and Multiline Messages
    - Handles embeds, attachments, reactions, replies
- Account Tracking & Away Notifications
    - Handles online status and cross-device account sharing
- Batches & Message IDs
    - Handles reactions, replies

The goal is to allow standard IRCv3 clients to be able to join and interact without ever needing the custom client, furthering adoption.

When the client negotiates the ability to do so, direct messages will be end-to-end encrypted with a symmetric key created via DH exchange, such that each person's DMs are encrypted between the two in a way the server cannot know, but the server can host the messages for later retrieval. This is what enables messaging someone who is offline, or reading chat history without saving the entire conversation to disk (later enabling multi-device support).

### Mumble

Mumble is an amazing VoIP solution, and we can leverage it to enable low-latency communication between users.

### `libobs`

OBS Studio is a popular program for streaming, screen sharing, and more. We can leverage the underlying library, `libobs`, to handle screen and video capture, and relay it all through WebRTC ([now supported](https://github.com/obsproject/obs-studio/commit/851a8c216e14617fb523951839f3bdb240e85141) as of 2023).

### WebRTC + coturn

WebRTC allows peer-to-peer communication, which is essential for low-latency communication and streaming. It also enables some interesting side functionality (see [WebTorrents](https://webtorrent.io/)). Optionally to the stack, a server can host coturn to allow certain users with less-than-ideal networking infrastructure to participate. This is how most, if not all, other video/screen/voice products work.

### Attachments Server

There are a handful of features that are necessary for a modern chat client:

- User file and image uploads
- Rich embed resolution/rendering
- User account profile photos, banners, etc.
- Custom emojis

This service will handle all of that.

### Control Server

A simple server with a lot of little purposes, it is the glue that makes the whole architecture work. This will be the majority of my efforts until there is enough to start building a client.

- Handle account creation and restrictions on users
- Monitor health and logs of other components
- Handle configuration settings for the server
- Relay account data between inspIRCd and Mumble
- Handle user online/away/offline statuses
- Handle authentication/authorization rules
- (later) Handle federation of user accounts across other control servers

### Client

I am primarily a web tech code monkey, so I'm going to (unfortunately) do Electron. In terms of portability it ranks highest of the popular options, so once something better comes along, I'll switch to that. The job of the client is to integrate the various services into one visually. It is an IRC client, a Mumble client, a WebRTC peer, etc.

## Later Plans

Once the base product is built, there are a few follow-up features I want to build out.

### WebTorrent File Sharing

Once the WebRTC infrastructure is in place, we can utilize WebTorrent tech to allow peer-to-peer file streaming. This would also allow users to send files larger than the server-configured limit, limited to direct messages.

### Server WebTorrent Sites

Users can create their own basic HTML/CSS sites like AngelFire/Geocities, advertise them from the server, and host them in a peer-to-peer way, with the server acting as a fallback peer if the user goes offline. The server could also host its own sites with the same tech.

### Content Streaming

If `libobs` is already going to be added to the stack, I can also add out streaming endpoint support, so you can do Twitch-style live streams using the actual OBS Studio. It could integrate directly into the app/stack, and feel seamless.

### Clip Hosting

Once WebTorrent, streaming, and broader communication is in place, you can then build out a basic YouTube/Bitchute clone.

### Search Aggregation

The search feature in Discord sucks, so once there is enough stuff in place, I can look into a proper search index for the stack.

