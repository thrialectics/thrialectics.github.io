---
title: "Naming Ceremonies//Digital Signatures"
date: 2025-09-08
---

Dear Friends,  
  
Have you thought much about the purpose of a signature?

I only have recently. There’s a paradox inherent in it. Your signature functions as a public mark of your private intent; it needs to be something you can perfectly reproduce every time you use it, but it should be irreproducible by anyone else.

Historically, the signature has been something performed with the body. It was a design that you committed to muscle memory and pressed onto paper.

Of course, a written signature is easy to forge. Its reproducibility is **precisely the** **attack surface**.

![Alan Turing Signature; this isn't important, don't worry.](https://substackcdn.com/image/fetch/,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fcd642c12-e649-43b4-8c1c-8c812985f1e0_640x129.png "Alan Turing Signature; this isn't important, don't worry.")

This problem of forgery has only intensified in our age of advanced digital reproducibility. Now the paradox has scaled: we have a crisis that is no longer about a forged signature on a single document, but a forged “self” across a global network.

### True Names//Digital Doppelgängers

Our oldest myths teach us that names are not trivial. In many traditions, to know a True Name is to have power over the named; to give one’s name is to become vulnerable. These stories encode the intuition: our names are contested sites of authorship, stewardship, and power.

Applying this intuition to our present context: today, at the gates of every digital platform, we are asked to surrender our identification. Given names, aliases, handles. By doing so, we receive in return access to services, goods, and affordances for connection.

But today’s platforms take this dynamic a step further: they not only take our names, but also rename us. The vast datasets that they collect allow for the creation of predictive models meant to represent our aggregate selves. These identities, legible only to the platform algorithms, become like our doppelgängers. We’re renamed not once, but endlessly. Every click, even every pause, is information that serves to build our double. The simulations are not crude, either; they can seem to anticipate us better than we anticipate ourselves.

Now, with large language models trained on our collective expression, the problem deepens. The style of your words and the rhythm of your speech become mimickable with an uncanny fidelity. In a world of algorithmic knowledge and perfect forgeries, how would we become stewards of our True Names, or prove authorship of our creations?

### Fictional Proxies//Narrative Sandboxing

This wrestling with the problems of the Name—of the negotiation of wanting and not wanting to be known—inspired me to think differently about some of my _Serpent/Dove_ designs.

I had an idea: to better understand some of these concepts, I could practice with fictional proxies. Like a narrative sandboxing.

I have a tool in development for trying this out, and once again I’m inviting you to step into the role of beta tester when it’s ready. It has two parts: one is for naming a character; the other is for claiming your authorship of it.

![A screenshot of the tool; the assigned alias is "Clio Star." This might be imporant.](https://substackcdn.com/image/fetch/,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fdb46809b-8d88-4125-a0e9-27ebea3115e2_1704x1004.png "A screenshot of the tool; the assigned alias is \"Clio Star.\" This might be imporant.")

#### **Part One: A Naming Ceremony**

First, take some time to imagine some character. It can be a fictive proxy for yourself, or nothing like you at all. Once you have the visualization, answer two questions:

1. _Your character wakes up with unlimited free time and resources for two weeks. What do they do?_
    
2. _When does your character feel most alive; most purely themselves?_
    

The responses are mapped into semantic space using OpenAI embeddings, which locate the meaning of your answers as coordinates. The system then semantically matches your answers to my database of names, adjectives, and nouns.

The results generate an **Alias** and a **Cryptonym**. The first is a given name, and the second is two words, styled as a handle.

#### **Part Two: Cryptographic Claiming**

Once your character is named, you’re given the means to claim authorship via a digital signature: the tool provides a twelve-word recovery phrase and generates your private key. With this key, you could later “sign” any action as this character. No one else could, unless they intercepted your key. If you keep it safe, your authorship remains provable; if you lose it, it can’t be restored (not even by me).

This tool will _not_ be adequate for verifying uniqueness; you could generate countless characters with it if you really wanted to. It’s more about a kind of practice through speculative play. With a name, a handle, and a digital signature, you steward both the story and your authorship.

If you’d like early access, have feedback on my methodologies, or would graciously volunteer your expertise to review the tool’s codebase and logic, please comment below or DM so I can send you a link when it’s ready.

Otherwise, stay tuned, and let me know if you have any feedback.  

So Long,  
  
Marianne 🩵

_  
P.S. Ironically, as I was writing this, there was a security breach in the NPM ecosystem (the JavaScript package registry that many web applications depend on). Malicious code specifically targeting cryptographic operations got pushed to packages that see billions of weekly downloads. This kind of risk is the reason I’ve chosen simple, low-stakes implementations to explore these ideas with you.  
  
I have much to learn; I’m sure I’ll learn a lot of it from many of you. xx_

[](https://substack.com/profile/1054810-matt-de-caussin)[](https://substack.com/profile/16477561-ramjet_oddity)[](https://substack.com/profile/328657456-chii)

3 Likes

[](https://substack.com/note/p-172917398/restacks?utm_source=substack&utm_content=facepile-restacks)
