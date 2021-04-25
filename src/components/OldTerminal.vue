<template>
    <div class="container">
        <div class="row">
            <div class="col-12">
                <div class="terminal m-2 p-3 bg-dark text-white position-relative text-start rounded" ref="terminalContainer">
                    <div class="terminal-history overflow-auto mb-3" ref="terminalHistory">
                        <div v-for="(history, index) in terminalHistory" :key="index" class="container">
                                <div v-if="history.files" class="row">
                                    <span v-for="file in history.files" :key="file" v-html="file" class="col-4"></span>
                                </div>
                                <div v-else>
                                    <span v-html="history.commandText"></span>
                                </div>
                        </div>
                    </div>
                    <div class="terminal-entry position-absolute bottom-0 left-0 pb-1">
                        <b class="text-primary">~/{{currentPath.text}}</b>$
                        <input type="text" v-model="terminalInput" ref="terminal_input" class="terminal-input bg-dark text-white border-0">
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'Terminal',

    data() {
        return {
            terminalInput: "",

            terminalHistory: [],

            currentPath: {
              text: "",
              path: [], 
            },
            currentDir: {},
            dirStructure: {
                "/": {
                    "home": {
                        "pictures": {
                            "picture1.png": "img",
                            "picture2.png": "img",
                            "picture3.png": "img",
                            "picture4.png": "img",
                            "picture5.png": "img",
                        },
                        "documents": {
                            "readme.txt": "Texto que está dentro desse arquivo",
                            "important.txt": "text",
                        }
                    },
                    "usr": {
                        "lib": {
                            "lib-javascript.so": "file"
                        },
                    }
                }
            },

            acceptedCommands: {
                "echo":  this.echoTerminal,
                "ls":    this.showDirFiles,
                "clear": this.cleanTerminal,
                "cd":    this.navigateTerminal,
                "mkdir": this.makeDirectory,
            }
        }
    },

    mounted() {
        this.focusTerminalInput();

        this.currentDir = this.dirStructure["/"]

        this.$refs.terminalContainer.addEventListener('click', () => { this.focusTerminalInput() });
    
        this.$refs.terminal_input.addEventListener('keypress', e => {
            if(e.charCode == 13)
            {
                let terminalInputSplited = this.terminalInput.trimLeft().split(' ')
                let commandText = terminalInputSplited[0];
                let commandArgs = terminalInputSplited.slice(1);
                let commandAction = this.acceptedCommands[commandText];

                if (!commandAction)
                {
                    this.terminalHistory.push({commandText: (commandText.length > 0  ? `command '${this.colorize(commandText, 'info')}' not recognized` : '')})
                    this.terminalInput = "";
                    return;
                }
                this.echoTerminal(this.colorize(this.currentPath.text+' >$ ','primary') + this.colorize(this.terminalInput, 'info'));
                commandAction(commandArgs)
                this.terminalInput = "";
            }
        })
    },

    methods: {
        colorize(text, color) {
            return `<span class="text-${color}">${text}</span>`;
        },


        focusTerminalInput() {
            this.$refs.terminal_input.focus()
        },

        echoTerminal(args = []) {
            this.terminalHistory.push({ commandText: args });
            let elem = this.$refs.terminalHistory;
            setTimeout(() => {
                elem.scrollTop = elem.scrollHeight;
            }, 50); 
        },

        showDirFiles() {
            let dir = this.currentDir
            let possibleKeys = Object.keys(dir);
            this.terminalHistory.push({ files: possibleKeys })
        },

        navigateTerminal(args = []) {
            let destinyPath = args[0];
            if(destinyPath == '..'){
                if(this.currentPath.path.length > 0){
                    //Alterando o objeto contendo as keys
                    this.currentDir = this.currentPath.path[this.currentPath.path.length-1];
                    this.currentPath.path.pop();
                    //Alterando o texto
                    let splitedPathText = this.currentPath.text.split('/');
                    splitedPathText.pop();
                    splitedPathText.pop();
                    this.currentPath.text = splitedPathText.join('/');
                }
            }else if(Object.keys(this.currentDir).includes(destinyPath)){
                //Alterando o objeto contendo as keys
                this.currentPath.path.push(this.currentDir);
                this.currentDir = this.currentDir[destinyPath];
                //Alterando o texto
                this.currentPath.text += destinyPath + "/";
            }
        },

        makeDirectory(args = {}) {
            let dirName = args[0]

            if (!dirName){
                this.echoTerminal({message: "mkdir dirname can't be empty"})
                return;
            }

            this.dirStructure[dirName] = {
                files: []
            }
        },

        cleanTerminal() {
            this.terminalHistory = []
        },


    }
}
</script>

<style scoped>


.terminal-input:focus {
    border: none;
    outline: none;
}

.terminal-history{
    height: 300px;
}

</style>