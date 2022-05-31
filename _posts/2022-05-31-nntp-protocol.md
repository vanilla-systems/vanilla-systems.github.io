---
layout: post
title: "The nntp protocol"
categories: misc
author: "Mr. M"
---

# Intro

The Network News Transfer Protocol (nntp) is an application protocol used for transfering Usenet articles between news servers, and for reading/posting articles by client applications. 

nntp is defined in multiple RFCs:

1. [RFC977 - The basic protocol - 1986](https://datatracker.ietf.org/doc/html/rfc977) 
2. [RFC3977 - Extensions to the protocol - 2006](https://datatracker.ietf.org/doc/html/rfc3977) 
3. [RFC4642 - NNTP over TLS - 2006](https://datatracker.ietf.org/doc/html/rfc4642) 

Focus here is on RFC977 and RFC3977

# RFC977 - The basic protocol commands

Each command line must be terminated by a CR-LF (Carriage Return - Line Feed) pair.
Command lines shall not exceed 512 characters in length, counting all characters including spaces, separators, punctuation, and the trailing CR-LF (thus there are 510 characters maximum allowed for the command and its parameters).

Responses are of two kinds, textual and status.

1xx - Informative message  
2xx - Command ok  
3xx - Command ok so far, send the rest of it.  
4xx - Command was correct, but couldn't be performed for some reason.  
5xx - Command unimplemented, or incorrect, or a serious program error occurred.  

The next digit in the code indicates the function response category.


x0x - Connection, setup, and miscellaneous messages  
x1x - Newsgroup selection  
x2x - Article selection  
x3x - Distribution functions  
x4x - Posting  
x8x - Nonstandard (private implementation) extensions  
x9x - Debugging output  


In general, 1xx codes may be ignored or displayed as desired;  code 200 or 201 is sent upon initial connection to the NNTP server 
depending upon posting permission; code 400 will be sent when the NNTP server discontinues service (by operator request, for example);
and 5xx codes indicate that the command could not be performed for some unusual reason.

Information are stored as articles in usenet. Each article consists out of a header and a body.

The header holds header information:
todo

The body contains the payload of the message:
todo


# ARTICLE
Display the header, a blank line, then the body (text) of the
specified article.  Message-id is the message id of an article as
shown in that article's header.

## Request
ARTICLE <message-id> \r\n

## Response(s)
220 n <a> article retrieved - head and body follow (n = article number, <a> = message-id)  
221 n <a> article retrieved - head follows  
222 n <a> article retrieved - body follows  
223 n <a> article retrieved - request text separately  
412 no newsgroup has been selected  
420 no current article has been selected  
423 no such article number in this group  
430 no such article found  

## Example Request
ARTICLE \<asdaasadasdasadsdsaasasd@nyuu\>

## Example Response
220 \r\n   
\<HEADER\> \r\n  
\<BODY\> \r\n  

# BODY
Same as ARTICLE command but only returns the body

# HEADER
Same as ARTICLE command but only returns the header

# STAT
Similar to ARTICLE command but the STAT command is similar to the ARTICLE command except that no text is returned.  When selecting by message number within a group, the STAT command serves to set the current article pointer without sending text. The returned acknowledgement response will contain the message-id, which may be of some value.  Using the STAT command to select by message-id is valid but of questionable value, since a selection by message-id does NOT alter the "current article pointer".

# GROUP
Query information about a given newsgroup

## Request
GROUP \<newsgroup\>

## Response(s)
211 n f l s group selected  
        (n = estimated number of articles in group,  
        f = first article number in the group,  
        l = last article number in the group,  
        s = name of the group.)  
411 no such news group  

## Example Request
GROUP alt.binaries.test

## Example Response
211 5629024605 266503 5629291107 alt.binaries.test

# HELP
Provides a short summary of commands that are understood by this
implementation of the server. The help text will be presented as a
textual response, terminated by a single period on a line by itself.

## Response(s)
100 help text follows

# IHAVE
IHAVE command in that it is intended for use in transferring already-posted articles between hosts.
Normally it will not be used when the client is a personal newsreading program (in contrast to the POST command).
In particular, this function will invoke the
server's news posting program with the appropriate settings (flags, options, etc) to indicate that the forthcoming article is being
forwarded from another host.

## Request
IHAVE \<messageid\>

## Response(s)
235 article transferred ok  
335 send article to be transferred.  End with \<CR-LF\>.\<CR-LF\>
435 article not wanted - do not send it  
436 transfer failed - try again later  
437 article rejected - do not try again  

# POST
Intended to post new articles to a news host.

## Response(s)
240 article posted ok  
340 send article to be posted. End with \<CR-LF\>.\<CR-LF\>  
440 posting not allowed  
441 posting failed  

# LAST
The internally maintained "current article pointer" is set to the previous article in the current newsgroup.  If already positioned at
the first article of the newsgroup, an error message is returned and the current article remains selected.

## Request
LAST

## Response(s)
223 n a article retrieved - request text separately  
           (n = article number, a = unique article id)  
412 no newsgroup selected  
420 no current article has been selected  
422 no previous article in this group  

# NEXT
The internally maintained "current article pointer" is advanced to the next article in the current newsgroup.

## Request
NEXT

## Response(s)
223 n a article retrieved - request text separately  
        (n = article number, a = unique article id)  
412 no newsgroup selected  
420 no current article has been selected  
421 no next article in this group  

# LIST
Returns a list of all valid newsgroups and associated information.

## Request
LIST

## Response(s)
215 list of newsgroups follows
n-times - group last first p

# NEWGROUPS
A list of newsgroups created since \<date and time\> will be listed in the same format as the LIST command.

# Request
NEWGROUPS date time \[GMT\] \[\<distributions\>\]
The date is sent as 6 digits in the format YYMMDD.  
Time must also be specified. It must be as 6 digits HHMMSS.  

The optional parameter "distributions" is a list of distribution groups, enclosed in angle brackets. If specified, the distribution
portion of a new newsgroup (e.g, 'net' in 'net.wombat') will be examined and only those new newsgroups which match will be listed. If more than
one distribution group is to be listed, they must be separated by commas within the angle brackets.

# Response(s)
231 list of new newsgroups follows

# NEWNEWS
A list of message-ids of articles posted or received to the specified newsgroup since "date" will be listed. The format of the listing will
be one message-id per line, as though text were being sent.  A single line consisting solely of one period followed by CR-LF will terminate
the list. 
Date and time are in the same format as the NEWGROUPS command.

## Request
NEWNEWS newsgroups date time \[GMT\] \[\<distribution\>\]

## Response(s)
230 list of new articles by message-id follows

# SLAVE
Indicates to the server that this client connection is to a slave server, rather than a user.

This command is intended for use in separating connections to single users from those to subsidiary ("slave") servers. It may be used to
indicate that priority should therefore be given to requests from this client, as it is presumably serving more than one person. It
might also be used to determine which connections to close when system load levels are exceeded, perhaps giving preference to slave
servers.  The actual use this command is put to is entirely implementation dependent, and may vary from one host to another. In
NNTP servers which do not give priority to slave servers, this command must nonetheless be recognized and acknowledged.

## Request
SLAVE

## Response(s)
202 slave status noted

# QUIT
The server process acknowledges the QUIT command and then closes the connection to the client.

## Request
QUIT

## Response
205 closing connection - goodbye!
