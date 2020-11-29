---
title: Tools
weight: 2
---
# Tools: Creating a K8S environment to be managed by CI/CD.
Using tools to create Infra and providing Content. The toolset will provide the build environment to setup the infra, this documentation will provide the content part. The Infra that empowering the Documentation platform together with the Docuemntation itself (Content) will be used and extended while learning DevOps.

## Create a Docker-Image containing a simple documentation-site template.
The Dockerfile used to create this project looks like this:
'''yaml
FROM alpine:3.12
WORKDIR /
EXPOSE 1313/tcp

RUN apk update && \
apk add --no-cache git hugo && \
hugo new site doclib && \
cd doclib && git init && \
git submodule add https://github.com/ethermol/hugo-book themes/book && \
cp -R themes/book/templateSite/content . && \
sed -i 's/My New Hugo Site/The DevOps Society/' config.toml && \
rm -rf /tmp/* && \
rm -rf /var/cache/apk/*

WORKDIR /doclib
CMD [ "hugo", "server", "--minify", "--theme", "book" ]
'''

The repo used to contain the site contents is added as a submodule. The site contents can be changed by changing the repo. The contents is managed using the markdown format.


## Content and Layout for the documentation-site will be stored in a separate repo.
Within the hugo-book Arepo execute the following commands to get things going:
'''bash
docker build -t hugo .
docker run --name hugo --network host hugo

docker exec -ti $(docker ps -l -q) /bin/sh
# Within Docker container:
cd /doclib/themes/book
git pull
cd ../..
cp -R themes/book/templateSite/content .
# Site will be updated and can be explored using http://localhost:1313/

'''
I thought "docker run -p 1313:1313 hugo" should also work but it doesn't probably because the domain names within the url:s are wrong. It was meant to publish to the <ip-addr>:1313 of the Docker-host.

## Adding color to code blocks (recognized formats)

| Pfx | Language                                                                                                                                                                                                  |
|-----|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| A   | ABAP, ABNF, ActionScript, ActionScript 3, Ada, Angular2, ANTLR, ApacheConf, APL, AppleScript, Arduino, Awk                                                                                                |
| B   | Ballerina, Base Makefile, Bash, Batchfile, BibTeX, BlitzBasic, BNF, Brainfuck                                                                                                                             |
| C   | C, C#, C++, Caddyfile, Caddyfile Directives, Cap'n Proto, Cassandra CQL, Ceylon, CFEngine3, cfstatement, ChaiScript, Cheetah, Clojure, CMake, COBOL, CoffeeScript, Common Lisp, Coq, Crystal, CSS, Cython |
| D   | D, Dart, Diff, Django/Jinja, Docker, DTD                                                                                                                                                                  |
| E   | EBNF, Elixir, Elm, EmacsLisp, Erlang                                                                                                                                                                      |
| F   | Factor, Fish, Forth, Fortran, FSharp                                                                                                                                                                      |
| G   | GAS, GDScript, Genshi, Genshi HTML, Genshi Text, Gherkin, GLSL, Gnuplot, Go, Go HTML Template, Go Text Template, GraphQL, Groovy                                                                          |
| H   | Handlebars, Haskell, Haxe, HCL, Hexdump, HLB, HTML, HTTP, Hy                                                                                                                                              |
| I   | Idris, Igor, INI, Io                                                                                                                                                                                      |
| J   | J, Java, JavaScript, JSON, Julia, Jungle                                                                                                                                                                  |
| K   | Kotlin                                                                                                                                                                                                    |
| L   | Lighttpd configuration file, LLVM, Lua                                                                                                                                                                    |
| M   | Mako, markdown, Mason, Mathematica, Matlab, MiniZinc, MLIR, Modula-2, MonkeyC, MorrowindScript, Myghty, MySQL                                                                                             |
| N   | NASM, Newspeak, Nginx configuration file, Nim, Nix                                                                                                                                                        |
| O   | Objective-C, OCaml, Octave, OpenSCAD, Org Mode                                                                                                                                                            |
| P   | PacmanConf, Perl, PHP, PHTML, Pig, PkgConfig, PL/pgSQL, plaintext, Pony, PostgreSQL SQL dialect, PostScript, POVRay, PowerShell, Prolog, PromQL, Protocol Buffer, Puppet, Python, Python 3                |
| Q   | QBasic                                                                                                                                                                                                    |
| R   | R, Racket, Ragel, react, ReasonML, reg, reStructuredText, Rexx, Ruby, Rust                                                                                                                                |
| S   | SAS, Sass, Scala, Scheme, Scilab, SCSS, Smalltalk, Smarty, Snobol, Solidity, SPARQL, SQL, SquidConf, Standard ML, Stylus, Swift, SYSTEMD, systemverilog                                                   |
| T   | TableGen, TASM, Tcl, Tcsh, Termcap, Terminfo, Terraform, TeX, Thrift, TOML, TradingView, Transact-SQL, Turing, Turtle, Twig, TypeScript, TypoScript, TypoScriptCssData, TypoScriptHtmlData                |
| V   | VB.net, verilog, VHDL, VimL, vue                                                                                                                                                                          |
| W   | WDTE                                                                                                                                                                                                      |
| X   | XML, Xorg                                                                                                                                                                                                 |
| Y   | YAML, YANG                                                                                                                                                                                                |
| Z   | Zig                                                                                                                                                                                                       |
## Build, Run and test the image locally.
## Create a multi-arch image and push it to Docker-hub.
## Within AWS create a t4g (ARM based) instance using cfn
## Install a single host K8S instance on the t4g-instance.
## Deploy the Docker-Instance using Helm.
## DevOps-2: CI/CD

