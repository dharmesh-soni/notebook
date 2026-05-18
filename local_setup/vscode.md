### settings.json
```json
{
    // Python
    // "python.analysis.typeCheckingMode": "strict", // off, basic strict; Note: can be set to specific files

    // Editor
    "editor.colorDecorators": false,
    "editor.indentSize": "tabSize",
    "editor.language.colorizedBracketPairs": [],
    // "explorer.decorations.colors": false, // not sure how does it work
    "editor.codeActionsOnSave": {

    
        // Organize the imports
        // "source.organizeImports": "explicit"
    },
    "editor.minimap.autohide": true,
    "editor.minimap.renderCharacters": false,

    // 
    "remote.extensionKind": {
        
        "pub.name": [
            "ui"
        ]
    },

    // Notebook
    "notebook.output.scrolling": true,
    "notebook.lineNumbers": "on",
    "notebook.output.wordWrap": true,

    // Git
    "git.openRepositoryInParentFolders": "never",
    
    // Github
    "githubPullRequests.pullBranch": "never",

    // TODO extension
    "todo-tree.general.statusBar": "total",
    "todo-tree.highlights.highlightDelay": 0,
    "todo-tree.highlights.defaultHighlight": {
        "icon": "alert",
        "type": "text",
        "foreground": "#F00",
        "opacity": 50,
        "iconColour": "#000000"
    },
    "todo-tree.general.tags": [
        "BUG",
        "HACK",
        "FIX",
        "TODO",
        "TEMP",
        "dTODO"
    ],
    "todo-tree.highlights.customHighlight": {
        "TODO": {
            "icon": "$(check)",
            // "iconColour": "editorError.foreground",
            // "type": "line",
            "type": "tag",
            "foreground": "#0000FF",
            // "foreground": "black",
            // "background": "black",
        },
        "BUG": {
            "icon": "$(bug)",
            "foreground": "#FF0000",
            "type": "tag",
        },
    },
    // "todohighlight.keywords":[
    //     {},
    // ]


    // "terminal.integrated.env.osx": {
    //     "PYTHONPATH": "${workspaceFolder}/src:${env:PYTHONPATH}"
    // },
    // "python.envFile": "${workspaceFolder}/.env",
    
    // AWS
    "aws.telemetry": false,

    // Azure
    "azure.resourceFilter": [
        "b0257c14-cacc-44c6-8927-5b4ce5de0874/fcd3510f-c0ef-44f9-bae3-68ec51417964"
    ],

    // Others
    "[xml]": {
        "editor.defaultFormatter": "redhat.vscode-xml",
        "editor.codeActionsOnSave": {},
    },
    "files.exclude": {
        "**/__pycache__": true,
        "**/*.pyc": true,
        "**/.pytest_cache": true,
        // "**/.*_cache": true,
    },
    "json.schemas": [

    ],
    "hediet.vscode-drawio.resizeImages": null,
    "chat.commandCenter.enabled": false,
    "github.copilot.enable": {
        "*": false,
        "plaintext": false,
        "markdown": false,
        "scminput": false
    },

    // Terminal
    // "terminal.integrated.fontSize": 12, 
    // "terminal.integrated.fontFamily": "Fira Code",

    // Source control
    // "scm.alwaysShowRepositories": true,

    // VS Code / workbench
    "workbench.editorAssociations": {
        "*.html": "default",
        "*.csv": "default"
    },
    "workbench.tree.indent": 15,
    "workbench.tree.renderIndentGuides": "always",

    "workbench.colorCustomizations": {
        // Tree Extension
        "tree.indentGuidesStroke": "#E50914", // "#05ef3c",
        "tree.inactiveIndentGuidesStroke": "#E7D4E8",

        // Terminal setting
        // "terminal.foreground" : "#00FD61",
        // "terminal.background" : "#383737",

        // "[Your Theme Name]": {
        // "list.inactiveSelectionBackground": "#434343",
        // }// "sta"


    },
    "workbench.activityBar.location": "top",
    // "workbench.statusBar.visible": false,
    

    "workbench.colorTheme": "GitHub Light Default",
    "editor.dropIntoEditor.preferences": [
    ],
    "workbench.settings.applyToAllProfiles": [
        
    ],

}
```