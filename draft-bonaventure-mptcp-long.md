---
title: Extending the Multipath TCP option space in the SYN segments 
abbrev: MPTCP long option
docname: draft-bonaventure-mptcp-long-00
date: 2016-09-30
cat: info

ipr: trust200902
area: Transport
wg: MPTCP Working Group
kw: Internet-Draft

coding: us-ascii 
stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
  -
     name: Olivier Bonaventure
     ins: O. Bonaventure
     organization: Tessares
     email: Olivier.Bonaventure@tessares.net

normative:
  RFC6824:
  RFC0793:

informative:
  I-D.boucadair-mptcp-plain-mode:
  I-D.ietf-mptcp-experience:
  RFC7410:
  RFC7323:
  RFC2018:
  RFC5482:
  RFC5925:
  RFC6994:
  RFC7413:
  RFC1071:
  I-D.ietf-mptcp-rfc6824bis:    
  I-D.ietf-tcpm-tcp-edo:
  WT-348:
    title: Hybrid Access for Broadband Network
    author:
        - ins: Broadband Forum
    date: 2014
    target: http://datatracker.ietf.org/liaison/1355/
  HotMiddlebox:
    title: Are TCP extensions middlebox-proof? 
    author:
      - ins: B. Hesmans
      - ins: F. Duchene
      - ins: C. Paasch
      - ins: G. Detal
      - ins: O. Bonaventure
    seriesinfo: HotMiddlebox13
    date: Dec. 2013
    target: http://inl.info.ucl.ac.be/publications/are-tcp-extensions-middlebox-proof

--- abstract

The extensibility of TCP depends on the utilisation of TCP options
in the extended header. Unfortunately, the length of the TCP
Data Offset field restricts the length of the TCP options
in a TCP packet. In this document, we propose a modification
to the MP\_CAPABLE option to enable Multipath TCP enabled hosts
to use longer TCP options inside the SYN segments. 

--- middle

Introduction    {#intro}
============

TCP was designed with extensibility in mind since {{RFC0793}} includes
support for TCP options whose utilisation can be negotiated during
the three-way handshake that is used to establish a connection.
A wide range of TCP options have been defined {{RFC7410}}
Those TCP options can be included inside the extended TCP header
whose length is specified by the Data Offset field of the TCP
header {{RFC0793}}.

Unfortunately, some TCP use a large number of bytes and
it becomes difficult to combine several of them inside the SYN
segments. For the TCP segments exchanged over an established
connection, the EDO {{I-D.ietf-tcpm-tcp-edo}} extension allows
to place them inside the segment payload. Unfortunately, 
EDO does not allow to extend the options space inside the SYN
segments.

In this document, we propose an extension to the MP\_CAPABLE
option that is used by Multipath TCP {{RFC6824}} to support
the utilisation of long TCP options during the three-way handshake
that creates the first subflow of a Multipath TCP connection. 
Our solution builds upon Multipath TCP and not regular TCP for two
reasons. First, there is a clear demand for supporting
longer TCP options in the SYN segments for at least
one important application of Multipath TCP {{I-D.boucadair-mptcp-plain-mode}}
{{WT-348}}.
Second, Multipath TCP already includes some middlebox interference
mitigation techniques and by changing the base Multipath TCP 
specification we can expect that middleboxes will be support
correctly the proposed extension when Multipath TCP becomes widely
deployed in the global Internet.


This document is organise as follows. We first summarise the key
elements of the proposed solution in section {{exec}}. Then
we provide a detailed description of our proposed modification to the
MP\_CAPABLE option in section {{option}}. We then discuss 
our proposed solution copes with different types of middlebox
interference in section {{mbox}}.


Executive summary {#exec}
-----------------

Multipath TCP is a major extension to TCP that has lead to the development
of new use cases that leverage its multipath capabilities 
{{I-D.ietf-mptcp-experience}}. One difficulty for some of these use
cases is that the TCP extedend header that is used to carry TCP options
has a limited size caused by the size of the Data Offset field of the
TCP header. A solution is being designed to increase the length of
the TCP extended header in TCP segments {{I-D.ietf-tcpm-tcp-edo}}, but
it cannot be used in the SYN segments that are exchanged during the
initial three-way handshake. One of the current use cases of
Multipath TCP expects to be able to include long TCP options
inside those SYN segments {{I-D.ietf-tcpm-tcp-edo}}. Another example
is that it is almost impossible to combine TCP-ADO {{RFC5925}}
with Multipath TCP.

We propose a simple extension to the MP\_CAPABLE option that is
used to negotiate the utilisation of Multipath TCP. Our extension
allows to reuse up to 1020 bytes of the payload of the SYN segments
to carry TCP options. It is composed of two parts :

 - an additional byte (Extended Options Length) 
   in the MP\_CAPABLE option containing the length
   of the SYN payload that carries TCP options and not user data
 - a new TCP Checksum option that contains an Internet checksum
   computed over all the TCP options carried in a SYN segment


This document proposes to divide a SYN segment that includes the
modified MP\_CAPABLE option in four parts as shown in 
{{figsegment}}. The first part is the 20 bytes TCP header. The
Data Offset field of this header indicates the boundary between
the Extended TCP header and the payload. The MP\_CAPABLE
option is placed in the Extended TCP header. It includes the
Extended Options Field that indicates, in four bytes word like the
Data Offset field, the number of bytes that carry TCP options at
the beginning of the payload. One of the TCP options included in
this payload is the TCP Checksum option. This option contains a 
Checksum of the TCP options that are placed inside the payload.
It is used to detect interference from middleboxes that modify
TCP segments without understanding Multipath TCP or the extension
defined in this document. The last part of the TCP segment are the
user data bytes passed by the user. Such user data can be included
in SYN segments when the TFO extension is used {{RFC7413}}. 

~~~~~~~~~
        +=============================+
        |                             |
        |                             |
        |                             | TCP
 +----- |Data Offset                  |
 |      |                             |
 |      +-----------------------------+
 |      |         MP_CAPABLE          | Extended 
 |   +--|ExOL |   Optionx ...         | TCP
 |   |  |                             | Header
 +-> |  +-----------------------------+
     |  | Optiony      |   Optionz    | TCP Payload
     |  | ...                         |
     |  |         TCP   Checksum      |    
     +->+-----------------------------+
        |         User data (opt.)    | TCP Payload
        |                             | (cont.)
        +=============================+
~~~~~~~~~
{: #figsegment title="A TCP segment containing the modified MP\_CAPABLE option and the TCP Checksum option"}


The modified MP\_CAPABLE option   {#option}
==============

The MP\_CAPABLE option is the TCP option that is used to establish
the initial subflow of a Multipath TCP connection. Its format has
been recently updated by the MPTCP working group while preparing the
standards-track version of the protocol. When used in the 
SYN segment, this option is four bytes long as shown in
{{figmpc}}.

{{I-D.ietf-mptcp-rfc6824bis}} 


~~~~~~~~~
                     1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+---------------+---------------+-------+-------+---------------+
|     Kind      |    Length     |Subtype|Version|A|B|C|D|E|F|G|H|
+---------------+---------------+-------+-------+---------------+
~~~~~~~~~
{: #figmpc title="The existing MP\_CAPABLE option"}


Our proposed modification is pretty simple. We extend the
MP\_CAPABLE option to five bytes instead of 4, the last byte
carries the extended options length, i.e. the number of bytes
at the beginning of the payload of the SYN segment that contain
TCP options and not user data. This length is encoded as a number
of four bytes words. This implies that the payload of a SYN segment
can contain up to 1020 bytes of TCP options. 

~~~~~~~~~
                     1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+---------------+---------------+-------+-------+---------------+
|     Kind      |    Length     |Subtype|Version|A|B|C|D|E|F|G|H|
+---------------+---------------+-------+-------+---------------+
| Ext. Opt. Len.|
+---------------+
~~~~~~~~~
{: #figempc title="The extended MP\_CAPABLE option"}

We also define a new TCP option, called the TCP Checksum option.
The format for this option is shown in {{figchecksum}}. It is
used to detect middlebox interference on the endhosts.


~~~~~~~~~
                     1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+---------------+---------------+---------------+---------------+
|     Kind      |    Length     |            Checksum           |
+---------------+---------------+---------------+---------------+

~~~~~~~~~
{: #figchecksum title="The Options Checksum TCP option"}


Transmission of a segment containing the Extended MP\_CAPABLE option {#send}
-----------------------------------------------


The modified MP\_CAPABLE option can only be included inside
SYN segments. It MUST be ignored in segments whose SYN flag
is reset. 

Conceptually, a Multipath TCP implementation should use
the Extended MP\_CAPABLE option whenever it needs to send TCP
options that require more space than the maximum length of the
Extended TCP header. The implementation should prepare a list 
of all the TCP options that need to be included
in the SYN segment. 


We consider four different types of TCP options :

- Type 1: TCP options that MUST be placed in the normal TCP
  extended header :

   - the MSS option defined in {{RFC0793}}. Many middleboxes expect
     that this option will be present in all SYN segments and some
     even add it. 
   - the WScale option defined in {{RFC7323}}. This option modifies
     the semantics of the window field of the TCP header and
     there is no way to recover from the removal of this option
     by a middlebox.
   - the MP\_CAPABLE option defined in {{RFC6824}}. This option
     contains the length of the extended header.
   - If padding is required to align the regular TCP options
     length to a multiple a four bytes, (possible padding)

- Type 2: TCP options that MAY be placed either in the normal TCP
  extended header or inside the payload of the SYN segment
  depending on the available space

   - the TCP timestamp option defined in {{RFC7323}}
   - the SACK permitted option defined in {{RFC2018}}
   - the TCP User timeout option defined in {{RFC5482}}
   - the TCP EDO option defined in {{I-D.ietf-tcpm-tcp-edo}}
   - the TCP Fast Open Option defined in {{RFC7413}}
   - the TCP AO option defined in {{RFC5925}}. This option allows to
     authenticate the TCP header and payloads. It has two modes of operation.
     One mode authenticates the entire header (including the options)
     and the other mode authenticates the header without the options. When
     the latter mode is used, the TCP options included in the SYN payload 
     MUST be considered as TCP options and thus not be included in the
     authentication (except the AO option itself as defined in {{RFC5925}}).
   - the Experimental TCP options defined in {{RFC6994}}
   
- Type 3: TCP options that MUST be placed in the payload of the SYN segment

   - the TCP Checksum option defined in this document
   - the plain mode option proposed in
     {{I-D.boucadair-mptcp-plain-mode}}
   - any option longer than 40 bytes
 


The following procedure should be used on the initiating host
when the size of the TCP options that must be placed in a SYN
segment is larger than the available space in the extended TCP header.
When assembling the TCP options to be placed in a SYN segment
for the first subflow of a Multipath TCP connection, the 
initiating host MUST first place in the normal extended header 
the options of Type 1, possibly with padding to preserve alignment. 
Then, if space permits, options of Type 2 are added to the extended 
header. Options of Type 3 MUST be placed in the payload of the SYN 
segment.

When there are options in the payload of the SYN segment, one of 
them (ideally the last one) MUST be the TCP checksum option
defined in this document. This option carries an Internet checksum
{{RFC1071}} computed over a pseudo-header that contains the
MP\_CAPABLE option (see {{figpseudoheader}}) followed by all the
TCP options that are included in the SYN payload.


~~~~~~~~~
                     1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+---------------+---------------+-------+-------+---------------+
|     Kind      |    Length     |Subtype|Version|A|B|C|D|E|F|G|H|
+---------------+---------------+-------+-------+---------------+
| Ext. Opt. Len.|    Zero       |     Zero      |     Zero      |
+---------------+---------------+---------------+---------------+
~~~~~~~~~
{: #figpseudoheader title="The pseudo-header used to compute the TCP
checksum option"}

The part of the payload of the SYN segments that is covered by the
Extended Options Length field of the MP\_CAPABLE option MUST be
considered as belonging to the extended TCP header by a TCP
implementation. This implies that the transmission of a SYN containing
such payload bytes does not advance the SND.NXT state variable. For the
same reason, the reception of a SYN segment containing such
payload bytes does not advance the RCV.NXT state variable.

Reception of segments containing the Extended MP\_CAPABLE option  {#processing}
---------------------------------------------


When receiving a SYN segment containing the extended
MP\_CAPABLE option, a TCP implementation should behave as follows.

1. If the Extended Data Offset field is set to zero, it MUST reply with
an MP\_CAPABLE option whose extended data offset field is also
set to zero. 

2. If the Extended Data Offset field is larger than zero, it MUST verify
that the payload length is larger or equal than the value of this field.
If not, the host MUST respond to the SYN segment with a RST segment
containing the Middlebox Interference reason code. If the payload
length is valid, it MUST then recompute the TCP checksum option. If there is
no TCP checksum option or if the computed value is incorrect, it
MUST respond with a RST segment containing the Middlebox Interference reason
code.

3. Once the length of the payload and the TCP Checksum option have been
validated, the host can process the other TCP options that are included
in the payload. It MUST advance RCV.NXT with the number of bytes that
carry real data in the payload (if any). The TCP options that are
covered by the Extended Data Offset field do not increase RCV.NXT. 


When processing a SYN+ACK segment containing the extended
MP\_CAPABLE option, a TCP implementation should behave as follows.

1. If the Extended Data Offset field is larger than zero, it MUST verify
that the payload length is larger or equal than the value of this field.
If not, the host MUST respond with a RST segment
containing the Middlebox Interference reason code and try to re-establish
the connection without using options in the SYN payload. If the payload
length is valid, it MUST then recompute the TCP checksum option. If there is
no TCP checksum option or if the computed value is incorrect, it
MUST respond with a RST segment containing the Middlebox Interference reason
code and try to re-establish the connection without placing TCP options
in the SYN payload. If the application requires the utilisation of such
options, then an error code should be returned to the application.

2. Once the length of the payload and the TCP Checksum option have been
validated, the host can process the other TCP options that are included
in the payload.

3. The host MUST verify that the acknowledgement number of the received
segment matches SND.NXT. If not, this indicates a possible middlebox
interference and the host MUST respond to the SYN+ACK with a RST
segment containing the Middlebox Interference reason code and attempt
to reestablish the connection without using long TCP options.


Examples {#examples}
--------

As an illustration, let us consider three examples showing a TCP
three-way handshake with different types of option. The first
example, figure 


~~~~~~~~~
--SYN| seq=x,                                   |
     | Ext. H. (8b): MSS, MP\_CAPABLE, no data  |----->

<- SYN+ACK| seq=y, ack=x+1, 
          | Ext. H. (8b): MSS, MP\_CAPABLE, no data |--

----ACK[seq=y+1]-------------------------------------->
~~~~~~~~~
{: #figtcpregular title="The regular TCP three way handshake"}


~~~~~~~~~

-SYN| seq=x,                                                |
    | Ext. H. (12b): MSS, MP\_CAPABLE(3), NOP, NOP, NOP     |
    | Payload (16b): Timestamp, Checksum, NOP, NOP, no data |->

<-SYN+ACK| seq=y, ack=x+1                                   |
         | Ext. H. (12b): MSS, MP\_CAPABLE(3), NOP, NOP, NOP|
         | Payload (16b): Timestamp, Checksum, NOP, NOP, no data|-


----ACK| seq=x+1, ack=y+1       ----------------------------->
~~~~~~~~~
{: #figtcpnodata title="The three way handshake with options in the payload
but no data"}



Example (with TFO) 

~~~~~~~~~
-SYN| seq=x,                                                  |
    | Ext. H. (12b): MSS, MP\_CAPABLE(3), NOP, NOP, NOP       |
    | Payload (26b): Timestamp, Checksum, NOP, NOP, data (10b)|->

<-SYN+ACK| seq=y, ack=x+11                                   |
         | Ext. H. (12b): MSS, MP\_CAPABLE(3), NOP, NOP, NOP |
         | Payload (17b): Timestamp, NOP, NOP, data(5)       |---

----ACK| seq=x+1, ack=y+6      |-------------------------------->
~~~~~~~~~
{: #figtcpdata title="The three way handshake with options and data in the payload"}


The extended MP\_CAPABLE option is intended to be used initially in networks where 
it is known that there are no on-path middleboxes that could interfere with its
correct operation. To verify that this property holds, the following procedure
is recommended.

The client creates a Multipath TCP connection with a server that is known to
support Multipath TCP. It is recommended to use as destination port
number the destination port number to would be used to interact with the server.
The SYN sent by the client includes the extended MP\_CAPABLE option with the 
Extended Options Length set to 0, i.e. there are no TCP options in the payload.
If the server accepts the connection, it MUST reply with a SYN+ACK that
also includes the MP\_CAPABLE option with the Extended Options Length set
to 0.

If client receives the MP\_CAPABLE option with the Extended Options Length
set to 0, it assumes that the path towards this server can support the 
utilisation of TCP options in the SYN payload. Otherwise, the client must
refrain from sending TCP options in the SYN payload towards this
destination address.


Interaction with middleboxes   {#mbox}
==============

There are various types of middleboxes that could interact with
the proposed modification. Following the methodology used
in {{HotMiddlebox}}, we discuss here how different middleboxes
behaviours could affect the proposed extension.


- removal of the MP\_CAPABLE option in the SYN or SYN+ACK segment

If the initiating host receives a SYN+ACK without the MP\_CAPABLE
option in response to a SYN that contained the MP\_CAPABLE option
and options in the SYN payload, it MUST tear down the connection
attempt by sending a RST segment containing the Middlebox
interference reason code {{I-D.ietf-mptcp-rfc6824bis}} 
and immediately try to reestablish
the connection without placing any option inside the payload of the
SYN. The release of the initial connection attempt should not be
exposed to the application. For the application, it should be considered
as a equivalent to a normal retransmission of a SYN segment. 


- removal of another TCP option inside the extended TCP header

If the option is removed from the SYN, it will not be received by the
server and will not be negotiated. The standard TCP procedure applies.
Note that is the option is removed from the SYN+ACK and not from the
SYN, this may cause problems for options such as WScale {{RFC7323}}
as explained in {{HotMiddlebox}}.

- removal of a TCP option inside the payload of the
  SYN segment

There are two different ways of removing a TCP option inside the
extended header that need to be considered. Some middleboxes remove
TCP options by replacing them with a sequence of NOP options. Other
middleboxes remove the TCP options and adjust the packet length. 
In both cases, the receiver will detect the modification thanks to the 
Options Checksum. It will then respond with a RST segment that
includes the Middlebox interference reason code.

- removal of the payload of the SYN segment

In this case, the receiver will detect a mismatch between the 
Extended Options Length field of the MP\_CAPABLE option
and the payload length. It will reply with a RST segment
containing the Middlebox Interference reason code.

- modification of the content of the SYN payload or 
  addition/removal of bytes in the SYN payload

If the modification affects the part of the SYN payload that
contains TCP option, it will be detected by the Options Checksum
and the receiver will reply with a RST segment containing
the Middlebox Interference reason code.


Security considerations  {#security}
========================

To be provided

IANA Considerations {#IANA}
=========

The IANA should reserve an option number for the proposed Options Checksum option. 


Conclusion {#conclusion}
=================

To be provided



--- back

