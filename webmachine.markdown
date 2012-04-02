# Overview: Developing REST webservices with Webmachine #

## About Me ##

      Tim McGilchrist
      @lambda_foo
      Ruby, Java and Erlang programmer.

## General Aproaches to web services ##

Pick your poison:
 * focus on heavy three-tier applications. Java?
 * hide the web behind language-specific constrtucts. Rails?
 * provide low-level access to HTTP

## What is Webmachine? ##

Webmachine is described as a REST toolkit. Developed by Basho, makers of the
Riak distributed database. (Which is sweet and you should check it out? Talk on
it another time?)

"Written in Erlang, Webmachine is an application layer that adds server-side
HTTP semantic awareness on top of the excellent bit-pushing and HTTP
syntax-management provided by mochiweb, " wiki.basho.com/Webmachine.html

It's not a full featured stack like Rails, Django or ChicagoBoss.
It doesn't provide things you'd normally expect like:
 - ORM / DB connectivity
 - HTML templating
 - security framework
 - opinions on how to structure your app, outside of a REST/Resource oriented interface.
 - newbie friendly screencasts on developing a Blog in X mins

Though you can find solutions to many of these things on github!

Webmachine is different, it applys strict HTTP semantics to Web Application
resources. ie it correctly uses HTTP verbs and HTTP status codes.

Unlike say Rails, it *strongly* encourages adherance to HTTP semantics.

## Why use Webmachine? ##

If you're the sort of person that likes:
 - full freedom over how your application is structured and constructed.
 - you don't want a framework doing "magic" behind your bat.
 - you want a real REST interface.
 - you don't want the typical CRUD app, ie no DB layer
 - you want to use Erlang

With all the JS client side frameworks, building your backend in whatever and us
JS/html.

## Install WebMachine ##

You'll need Erlang, MacPorts, Homebrew or whatever your poison

    git clone git://github.com/basho/webmachine


http://wiki.basho.com/Webmachine-Quickstart.html

## In the beginning there were Resources. ##

Webmachine focus' on serving Resources so lets start there.

{codeblock lang:erlang }
-module(wm_slide_resource).

-export ([init/1,
          allowed_methods/2,
          to_html/2]).

-include_lib("webmachine/include/webmachine.hrl").

-record(context, {}).

init([]) ->
    {ok, #context{}}.

allowed_methods(ReqData, State) ->
    {['HEAD', 'GET'], ReqData, State}.

to_html(ReqData, State) ->
    HtmlDoc = "<html><body>" ++
              "Hi everybody" ++
              "</body></html>",
    {HtmlDoc, ReqData, State).

{endcodeblock}

1. allowed_methods restricts this resource to HEAD and GET. anything else will
return

    HTTP/1.1 405 Method Not Allowed

2. to_html means we only respond to text/html content types

    >Host: localhost:8000
    >Accept: image/jpeg
    < HTTP/1.1 406 Not Acceptable

3. We didn't have to write  much and we got content type negotiation, HTTP
status codes.

## Making it visible ##

 Webmachine uses dispatch / routing as you'd expect.

 Defined in <app>/priv/dispatch.conf

     {[], sm_web_resource, []}.
     {["slide", id], sm_resource_slide, []}.
     {['*'], sm_resource_static, []}.


 1. Provides the default root url, requests to localhost:8000/ will get served
from here.

2. /slide/[:key] will provide a slide identified by :key

3. Matches everything else. I've got it setup to serve the static resources for
my app like JavaScript, CSS and images.


## sm_web_resource: The Default ##

{ codeblock lang:erlang }

%% Module, export and include

allowed_methods(ReqData, State) ->
    {['HEAD', 'GET'], ReqData, State}.

to_html(ReqData, State) ->
    {ok, Content} = index_dtl:render([]),
    {Content, ReqData, State}.

{ endcodeblock }

Only accessible via GET or HEAD and only provides html.

What is this index_dtl:render()?

Templating solution that I'm using, it's a port of Django templates to Erlang
http://github.com/evanmiller/erlydtl

Similar to other templating languages erb, jsp, ??

 ---> Show index.dtl

 I'm not really using it for much templating but it's nicer than mixing raw HTML
 into your Erlang.



 Resources:
 http://steve.vinoski.net/pdf/IC-Build_Your_Next_Web_Application_with_Erlang.pdf
 http://steve.vinoski.net/pdf/IC-Wriaki.pdf
 http://steve.vinoski.net/pdf/IC-Developing_RESTful_Web_Services_with_Webmachine.pdf
 https://github.com/evanmiller/erlydtl
