---
layout: post
title:  "Talk to the language server"
date:   2023-09-18 09:00:00 -0400
categories: vscode java lsp language server
---

In this [issue](https://github.com/eclipse/eclipse.jdt.ls/issues/2313) someone was attempting to launch the Java language server (JDT-LS) (eg. used by VS Code Java) from the command-line, and communicate with it via. simple socket stream. While I provided a solution for how to do this, and even debug issues, I never really went through a proper example.

Part of the reason is that the `initialize` message that begins communication withe language server is complicated, and not pleasant to understand. There's a lot of options to configure for a language server (eg. capabilities, dynamic methods, settings) and it isn't as clean as it could be. However, to show a working language server, a lot those options can be removed. We don't need to show anything that complicated to have an interesting example.

I wanted to challenge myself to come up with an example that is short and easy to set up.

```
CLIENT_HOST=127.0.0.1 CLIENT_PORT=5036 ./org.eclipse.jdt.ls.product/target/repository/bin/jdtls --jvm-arg=-Dsocket.stream.debug=true --jvm-arg=-Dosgi.dev
```

So what does this do ? It starts the language server and has it listening on port `5036` of the location interface for connections. The additional JVM arguments (`socket.stream.debug=true`, `osgi.dev`) ensure language server will be listening for connections.

You might wonder, *"Why have such a complicated way for the language server do something so basic ?"*. If not, skip this paragraph. Basically, most clients will have started before the language server, so it's often easier for the clients to start as a "server" and have the language server initiate the connection with the client. In other words, the client is the "server" and the server is the "client". It's a bit counter-intuitive to speak of a client/server with the roles reversed in the context of connection establishment, so for this example we just pass some extra arguments to make it more intuitive. The language server supports establishing a connection both ways though.

Once the server has started our "client" simply connects with :

`socat - tcp:localhost:5036`

If we look at the [Language Server Protocol Specification](https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/#baseProtocol) we can get some hints on how communication should work. We can initiate using a message as simple as :

```json
Content-Length: 191\r\n
\r\n
{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"initializationOptions":{"workspaceFolders":["file:///home/rgrunber/git/lemminx"],"settings":{"java":{"autobuild":{"enabled":true}}}}}}
```

For ease of visualization, the content part is just :

```json
{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "initialize",
    "params": {
        "initializationOptions": {
            "workspaceFolders": [
                "file:///home/rgrunber/git/lemminx"
            ],
            "settings": {
                "java": {
                    "autobuild": {
                        "enabled": true
                    }
                }
            }
        }
    }
}
```

It's important to note that while the message header(s) (eg. `Content-Length`) are terminated by `\r\n`, the content part is `null`-terminated, so our client (eg. `socat`) needs a way to denote that. In bash, this can be done by `^@` (<kbd>ctrl</kbd>+<kbd>@</kbd>) followed by carriage return (<kbd>enter</kbd>). While this is technically the correct way, JDT-LS does seem to tolerate the `\r\n` termination for content, though it will throw a rather annoying error like :

```
Sep. 09, 2023 2:16:38 P.M. org.eclipse.lsp4j.jsonrpc.json.StreamMessageProducer fireError
SEVERE: Missing header Content-Length in input "

"
java.lang.IllegalStateException: Missing header Content-Length in input "

"
	at org.eclipse.lsp4j.jsonrpc.json.StreamMessageProducer.listen(StreamMessageProducer.java:91)
	at org.eclipse.lsp4j.jsonrpc.json.ConcurrentMessageProcessor.run(ConcurrentMessageProcessor.java:113)
	at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:539)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1136)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:635)
	at java.base/java.lang.Thread.run(Thread.java:833)
```

So putting all of this together, here is the full interaction from the client-side :

```bash
$ socat - tcp:localhost:5036
Content-Length: 191

{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"initializationOptions":{"workspaceFolders":["file:///home/rgrunber/git/lemminx"],"settings":{"java":{"autobuild":{"enabled":true}}}}}}^@
Content-Length: 93

{"jsonrpc":"2.0","method":"language/status","params":{"type":"Starting","message":"Init..."}}Content-Length: 118

{"jsonrpc":"2.0","method":"language/status","params":{"type":"Starting","message":"0% Starting Java Language Server"}}Content-Length: 2384

{"jsonrpc":"2.0","id":1,"result":{"capabilities":{"textDocumentSync":{"openClose":true,"change":2,"save":{"includeText":true}},"hoverProvider":true,"completionProvider":{"resolveProvider":true,"triggerCharacters":[".","@","#","*"," "]},"signatureHelpProvider":{"triggerCharacters":["(",","]},"definitionProvider":true,"typeDefinitionProvider":true,"implementationProvider":true,"referencesProvider":true,"documentHighlightProvider":true,"documentSymbolProvider":true,"workspaceSymbolProvider":true,"codeActionProvider":true,"codeLensProvider":{"resolveProvider":true},"documentFormattingProvider":true,"documentRangeFormattingProvider":true,"documentOnTypeFormattingProvider":{"firstTriggerCharacter":";","moreTriggerCharacter":["\n","}"]},"renameProvider":{"prepareProvider":true},"foldingRangeProvider":true,"executeCommandProvider":{"commands":["java.project.import","java.navigate.openTypeHierarchy","java.project.resolveStackTraceLocation","java.edit.handlePasteEvent","java.edit.stringFormatting","java.project.getSettings","java.project.resolveWorkspaceSymbol","java.project.upgradeGradle","java.project.createModuleInfo","java.edit.organizeImports","java.project.refreshDiagnostics","java.project.removeFromSourcePath","java.project.listSourcePaths","java.project.getAll","java.reloadBundles","java.project.isTestFile","java.project.getClasspaths","java.navigate.resolveTypeHierarchy","java.edit.smartSemicolonDetection","java.project.updateSourceAttachment","java.decompile","java.protobuf.generateSources","java.project.resolveSourceAttachment","java.project.addToSourcePath","java.completion.onDidSelect"]},"workspace":{"workspaceFolders":{"supported":true,"changeNotifications":true}},"typeHierarchyProvider":true,"callHierarchyProvider":true,"selectionRangeProvider":true,"semanticTokensProvider":{"legend":{"tokenTypes":["namespace","class","interface","enum","enumMember","type","typeParameter","method","property","variable","parameter","modifier","keyword","annotation","annotationMember","record","recordComponent"],"tokenModifiers":["abstract","static","readonly","deprecated","declaration","documentation","public","private","protected","native","generic","typeArgument","importDeclaration","constructor"]},"range":false,"full":{"delta":false},"documentSelector":[{"language":"java","scheme":"file"},{"language":"java","scheme":"jdt"}]},"inlayHintProvider":true}}}Content-Length: 119

{"jsonrpc":"2.0","method":"language/status","params":{"type":"Starting","message":"30% Starting Java Language Server"}}Content-Length: 162

{"jsonrpc":"2.0","method":"language/status","params":{"type":"Starting","message":"80% Starting Java Language Server - Opening \u0027org.eclipse.lemminx\u0027."}}Content-Length: 93

{"jsonrpc":"2.0","method":"language/status","params":{"type":"ProjectStatus","message":"OK"}}Content-Length: 167

{"jsonrpc":"2.0","method":"language/status","params":{"type":"Starting","message":"100% Starting Java Language Server - Refreshing \u0027/org.eclipse.lemminx\u0027."}}Content-Length: 90

{"jsonrpc":"2.0","method":"language/status","params":{"type":"Started","message":"Ready"}}Content-Length: 167

{"jsonrpc":"2.0","method":"language/status","params":{"type":"Starting","message":"100% Starting Java Language Server - Refreshing \u0027/org.eclipse.lemminx\u0027."}}
```

After initialization we can start making other requests like :

```json
{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "workspace/symbol",
    "params": {
        "query": "xmllanguage"
    }
}
```

which we send as :

```json
Content-Length: 85\r\n
\r\n
{"jsonrpc":"2.0","id":1,"method":"workspace/symbol","params":{"query":"xmllanguage"}}^@
```

and the server responds with :

```json
Content-Length: 1269
{"jsonrpc":"2.0","id":1,"result":[{"name":"XMLLanguageClientAPI","kind":11,"location":{"uri":"file:///home/rgrunber/git/lemminx/org.eclipse.lemminx/src/main/java/org/eclipse/lemminx/customservice/XMLLanguageClientAPI.java","range":{"start":{"line":27,"character":17},"end":{"line":27,"character":37}}},"containerName":"org.eclipse.lemminx.customservice"},{"name":"XMLLanguageService","kind":5,"location":{"uri":"file:///home/rgrunber/git/lemminx/org.eclipse.lemminx/src/main/java/org/eclipse/lemminx/services/XMLLanguageService.java","range":{"start":{"line":69,"character":13},"end":{"line":69,"character":31}}},"containerName":"org.eclipse.lemminx.services"},{"name":"XMLLanguageServer","kind":5,"location":{"uri":"file:///home/rgrunber/git/lemminx/org.eclipse.lemminx/src/main/java/org/eclipse/lemminx/XMLLanguageServer.java","range":{"start":{"line":83,"character":13},"end":{"line":83,"character":30}}},"containerName":"org.eclipse.lemminx"},{"name":"XMLLanguageServerAPI","kind":11,"location":{"uri":"file:///home/rgrunber/git/lemminx/org.eclipse.lemminx/src/main/java/org/eclipse/lemminx/customservice/XMLLanguageServerAPI.java","range":{"start":{"line":26,"character":17},"end":{"line":26,"character":37}}},"containerName":"org.eclipse.lemminx.customservice"}]}
```

which is just :

```json
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": [
        {
            "name": "XMLLanguageClientAPI",
            "kind": 11,
            "location": {
                "uri": "file:///home/rgrunber/git/lemminx/org.eclipse.lemminx/src/main/java/org/eclipse/lemminx/customservice/XMLLanguageClientAPI.java",
                "range": {
                    "start": {
                        "line": 27,
                        "character": 17
                    },
                    "end": {
                        "line": 27,
                        "character": 37
                    }
                }
            },
            "containerName": "org.eclipse.lemminx.customservice"
        },
        {
            "name": "XMLLanguageService",
            "kind": 5,
            "location": {
                "uri": "file:///home/rgrunber/git/lemminx/org.eclipse.lemminx/src/main/java/org/eclipse/lemminx/services/XMLLanguageService.java",
                "range": {
                    "start": {
                        "line": 69,
                        "character": 13
                    },
                    "end": {
                        "line": 69,
                        "character": 31
                    }
                }
            },
            "containerName": "org.eclipse.lemminx.services"
        },
        {
            "name": "XMLLanguageServer",
            "kind": 5,
            "location": {
                "uri": "file:///home/rgrunber/git/lemminx/org.eclipse.lemminx/src/main/java/org/eclipse/lemminx/XMLLanguageServer.java",
                "range": {
                    "start": {
                        "line": 83,
                        "character": 13
                    },
                    "end": {
                        "line": 83,
                        "character": 30
                    }
                }
            },
            "containerName": "org.eclipse.lemminx"
        },
        {
            "name": "XMLLanguageServerAPI",
            "kind": 11,
            "location": {
                "uri": "file:///home/rgrunber/git/lemminx/org.eclipse.lemminx/src/main/java/org/eclipse/lemminx/customservice/XMLLanguageServerAPI.java",
                "range": {
                    "start": {
                        "line": 26,
                        "character": 17
                    },
                    "end": {
                        "line": 26,
                        "character": 37
                    }
                }
            },
            "containerName": "org.eclipse.lemminx.customservice"
        }
    ]
}
```

This is a simple example, and there's a lot of options being left out, but it shows what's really happening between the client and server when they communicate.
