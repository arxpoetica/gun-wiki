This is the very first public document about AXE and is intended as a prototype draft for developers to provide feedback on the architecture. This page is not ready for general readership though, and will be entirely rewritten in the future.

## Cartoon Whitepaper

Yes, we're notorious for our comic strip explainers for everything - our [conflict resolution algorithm](https://gun.eco/distributed/matters.html), [how cryptography works](https://gun.eco/explainers/data/security.html), and more!

And yes, this is **mandatory reading** before you read the actual whitepaper. Check it out here:

http://gunDB.io/trusting/strangers.html

## Lightning Talk

Here is a 5 minute lightning talk that I had the honor of doing before Bram Cohen, Inventor of BitTorrent, at Berkeley.

It focuses on providing a brief explanation of how the **Proof of Propagation** algorithm works.

It is highly recommended you watch it before reading the whitepaper. 

https://www.facebook.com/BerkeleyBlockchain/videos/2006069823011271/ (starts at 12:26)

## Whitepaper

The actual whitepaper is hosted at Stanford, here:

https://www.stanford.edu/~nadal/A-Decentralized-Data-Synchronization-Protocol.pdf

However, we will be moving it to our new website when we launch it and officially hosting it there for future reference.

It is split into 2 papers, the first on AXE itself, the second on how GUN works. Why? Because what we claim to do in AXE is impossible unless you know how GUN's CRDT algorithm works. These two things unfortunately cannot be separated, especially since many people will hear about AXE without knowing about what GUN is.

The whitepaper is in a review draft, and may be inaccurate or incomplete in areas. We would greatly appreciate **pull requests** on the paper, especially little things like filling in the citation links, and stuff like that.

I have answered a ton of follow up questions already in person and over email, they mostly repeat a lot of information that is already in the paper, however they also provide a lot of valuable nuanced details that the paper does not go into. When I have time, I hope to dump those questions and answers into this wiki page, and then it would be useful for the community to help merge them into the official whitepaper so everything is in one place.

## Contributing

Pull requests would be greatly appreciated (see comments above), however the whitepaper itself is in a google document form. So here is how to contribute a pull request:

1. Open a PR or issue with "AXE:" as the title.
2. Describe which Section & Sub-section you are editing. (Not page number!)
3. Provide your changes as a whole paragraph or paragraphs. (Clearly marked start/finish)
4. It would also be nice to provide what the previous version was (clearly marked start/finish) to prevent any conflicting simultaneous PR/changes with others.

Your contributions will be recognized in the paper in an acknowledgements section.

In the future we will probably (if needed) migrate the paper PR/issues into its own repo.

## Code

[SEA](https://hackernoon.com/so-you-want-to-build-a-p2p-twitter-with-e2e-encryption-f90505b2ff8) is a necessary primitive to AXE, and it is already in alpha.

We have working demonstrations of AXE's identity/wallet system running on a fully P2P social networking dApp and automated load tests (using [PANIC](https://github.com/gundb/panic-server)) for it across a physical test network here:

https://youtu.be/C3akdQJs55E

Obviously, there is a lot more work to be done though. So I apologize in advance that this section is somewhat sparse other than showing that demonstrations of AXE already exist and work (this is important, since most cryptocurrencies are just vaporeware), but we will update docs with more details later.

Additionally, it is fun to note that we're recognized by GitHub as [#2 in Blockchain](https://github.com/topics/blockchain)!

Please join the [chatroom](https://gitter.im/amark/gun) to discuss things further! Cheers.