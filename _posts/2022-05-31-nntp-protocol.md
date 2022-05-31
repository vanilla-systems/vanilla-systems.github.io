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
## Response
220 n <a> article retrieved - head and body follow (n = article number, <a> = message-id)  
221 n <a> article retrieved - head follows  
222 n <a> article retrieved - body follows  
223 n <a> article retrieved - request text separately  
412 no newsgroup has been selected  
420 no current article has been selected  
423 no such article number in this group  
430 no such article found  
## Example Request
ARTICLE <<asdaasadasdasadsdsaasasd@nyuu>>

## Example Response
220 \r\n  
<<HEADER>> \r\n  
<<BODY>> \r\n  

# BODY
Same as ARTICLE command but only returns the body

# HEADER
Same as ARTICLE command but only returns the header

# STAT
Similar to ARTICLE command but the STAT command is similar to the ARTICLE command except that no text is returned.  When selecting by message number within a group, the STAT command serves to set the current article pointer without sending text. The returned acknowledgement response will contain the message-id, which may be of some value.  Using the STAT command to select by message-id is valid but of questionable value, since a selection by message-id does NOT alter the "current article pointer".

# GROUP
Query information about a given newsgroup

## Request
GROUP <<newsgroup>>

## Response
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

## Response
100 help text follows