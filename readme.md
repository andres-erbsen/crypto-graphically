I experimented some with representing cryptographic computations and
protocols graphically, with notation similar to electrical engineering
or digital signal processing circuits. This example will walk through
one execution of a simple one-way-authenticated key exchange protocol
and demonstrate that an execution of this form preserves confidentiality.

First, here is the (completely arbitrary) notation I will be using in
the diagrams.

- Randomness, the source of all power of cryptography, is represented
  using a star. Of course, there is a difference between drawing two
  arrows from the same random value (the same star) and combining two
  different random samples. The graph form encodes this naturally.
- I to use The Cloud to represent to adversary. If you prefer, imagine
  that it is full of horned red creatures or shady figures with black
  hats. Vertical lines below the cloud represent the information it
  has access to before outputting a particular message. This naturally
  encodes the "arrow of time", differentiating between a process that
  first verifies a signature and then reveals the signing key from one
  that does the opposite.
- When The Cloud is being good, any message from it is simply a copy
  of the latest message sent to it from the other side. Of course, our
  reasoning about the protocol does not assume anything that naive,
  but it may be helpful for understanding the intended execution of
  the protocol.
- Functions are represented by boxes, with special kind of boxes for
  for tupling/untupling. "More special" inputs are on the side by
  convention: for mux, the choice of which top input to use comes from
  the side; for functions that have secret and public inputs the
  secret input is on the side.
- A horizontal dashed line with a o-arrow going into it has the
  following semantics. If the input is true in some execution, the
  graph is executed as if the line wasn't there. If the input is
  false, all values created *below* the dashed line are replaced with
  dummy values (i.e., the protocol is aborted).
- ∃ stands for a value that is existentially quantified at the
  beginning of the lemma (as the big-O constant in complexity
  analysis).

![](svg/0-legend.svg)

The next image contains the protocol execution we will be
analyzing. The protocol is a subset of the
[SIGMA](http://webee.technion.ac.il/~hugo/sigma-pdf.pdf) authenticated
key agreement protocol, and perhaps the minimal key agreement protocol
built from Diffie-Hellman and Signatures that provides some (although
definitely not all) relevant security properties. The execution we are
analyzing is the canonical one: the two parties receive the same
number of inputs as if there was not an adversary, but these inputs
are controlled by the adversary.  After the shared key has been
established, the initiator of this execution proceeds to encrypt a
message using the shared secret. We will seek to derive that this
message remains confidential.

The following convention is used for symbols: capital letters
represent public keys and corresponding lowercase letters represent
corresponding secret keys. Furthermore, primed keys are short-term,
held by the same party that holds the corresponding unprimed key.

In arrows-and-messages notation, this protocol might be described as follows:

        —> gᵃ
        <— gᵇ, sig_B(gᵃ, gᵇ)
        —> E(gᵃᵇ, m)

![](svg/1-receiver-auth.svg){ width=100% }

Now, how are we going to establish that this protocol indeed provides
confidentiality for the message? We will use standard computational game-based
security definitions (e.g., [https://crypto.graphics/signature/](EUF-CMA)).
However, we are going to *first* transcribe the security definitions to
circuit format. This would be trivial for a two-worlds-style security definition
that completely characterizes the behavior of the primitive in question.
Single-experiment definitions need to be converted to two-worlds style first,
and then existentially quantified over behavior in the undefined cases.
For signatures, we get the following computation indistinguishability statement:
if a secret key has been only used for signing, and verify with the
corresponding public key returns "valid", the message must be one of the ones
signed with that key.  Of course, if verify returns "invalid", the message from
the adversary may be anything at all. Either way, we do not get any guarantees
about the contents of the signature.

![](svg/2-sign.svg){ width=100% }

The choice between which side of the adversary the signing and verification are
drawn does not have semantic significance. However, it is best to stop and think
about the quantifiers in this statement for a moment. The adversary represents
universal quantification: the two sides must be indistinguishable for any
polynomial-time (in the implicit security parameter) probabilistic program run
in the cloud. Furthermore, the adversary is the same on the left and on the
right -- and it must behave the same as it cannot tell which side is executed.
However, the behavior of the simplified model on the right is just hidden
convenience: the function marked with ∃ figures out whether the adversary
returned a forgery or a previously signed message, and sets the select inputs to
the muxes accordingly. In principle, that definition could have been inlined to
the lemma statement, but it would not have been useful for anything and would
have cluttered the display. Thus, we shall be satisfied with just the fact that
such a decoder always exists. The entire experiment is also quantified over the
number of signing queries (we will discuss polynomial-time bounding issues
later).

Now we will seek to apply this lemma to our protocol execution. First, let's
separate out all the signing-related parts from the rest of the protocol. In the
next picture we have the same computation graph as before, laid out suggestively
using knowledge about the sign-verify lemma.

![](svg/3-receiver-auth-sign.svg){ width=100% }

The long dashed vertical lines separate the signing part from the rest of the
protocol. For the purposes of the sign-verify lemma, the rest of the protocol
may as well be adversarial, and it would still hold. I.e., we fill the universal
quantifiers in the sign-verify lemma with a combination of the protocol
execution and the adversary interfering with it. After replacing the left-hand
side of the sign-verify lemma with the right-hand-side, we arrive at the
following:

![](svg/4-receiver-auth-rewritten.svg){ width=100% }

![](svg/5-case.svg){ width=100% }

![](svg/6-invariant.svg){ width=100% }

![](svg/7-shortcircuit.svg){ width=100% }

![](svg/8-rearrange-for-dh.svg){ width=100% }

![](svg/9-ddh.svg){ width=100% }

![](svg/99-victory.svg){ width=100% }
