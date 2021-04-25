<template>
    <div class="container">
        <div class="row">
            <div class="col-12">
                <div class="terminal m-2 p-3 bg-dark text-white position-relative text-start rounded" ref="DOMdivContainer">
                    <div class="terminal-history overflow-auto mb-3" ref="DOMdivHistory">
                        <div v-for="(h, mainIndex) in history" :key="'history_'+mainIndex" class="container">
                            <TerminalHistory :history="h"></TerminalHistory>
                        </div>
                    </div>
                    <div class="terminal-entry position-absolute bottom-0 left-0 pb-1">
                        <b class="text-primary">{{currentPath.text}}</b>$
                        <input type="text" v-model="inputConsole" ref="DOMinputTerminal" class="terminal-input bg-dark text-white border-0">
                    </div>
                </div>
            </div>
        </div>
        <div class="row text-start">
            <div class="col-12">
                <h4 class="mt-3">Commands</h4>
                <p v-for="(command, commandName) in validCommands" :key="'valid_command_'+commandName">
                    {{commandName}} - <i class="text-muted">{{command.description}}</i>
                </p>
            </div>
        </div>
    </div>
</template>

<script>
import TerminalHistory from './TerminalHistory'

export default {
    name: 'Terminal',

    components: {
      TerminalHistory
    },

    data() {
        return {

            absolutePath: {
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
            },

            currentPath: {
                text: "",
                pathObj: {},
                pathHistory: []
            },

            history: [
                /*{ path: "~/", type: "unique", value:"ls teste" },
                { path: "~/", type: "list",   value:["dir01", "dir02", "dir03"] },*/
            ],

            inputConsole: "",

            validCommands: {
                "echo": {
                    description:    "Prints message",
                    function:       this.commandEcho
                },
                "clear":{
                    description:    "Clears the console",
                    function:       this.commandClear
                },
                "ls":{
                    description:    "List all the paths and files in the directory",
                    function:       this.commandList
                },
                "mkdir": {
                    description:    "Create a new directory",
                    function:       this.commandMakeDiretory
                },
                "cd":{
                    description:    "Move to the directory",
                    function:       this.commandMoveTo
                },
                "touch":{
                    description:    "Create new file",
                    function:       this.commandTouch
                },
                "cat":{
                    description:    "Print file content",
                    function:       this.commandCat
                }
            }
        }
    },

    mounted() {
        this.currentPath.pathObj    = this.absolutePath;
        this.currentPath.text       = "~/";
        this.$refs.DOMinputTerminal.addEventListener("keypress", e => {
            if(e.charCode == 13){
                this.executeCommand();
                this.inputConsole = "";
            }
        });
    },

    methods: {

        // Helpers
        emitMessage(type, message){
            message = type != 'normal' ? `<span class="text-${type}">` + message + '</span>' : message;
            this.history.push({
                path: this.currentPath.text,
                type: 'unique',
                value: message
            });
        },

        // Retorna o NovoCaminho se for válido, senão retorna o atual
        isPathValid(destinyPath, startingPoint=this.currentPath){
            var deepness    = 0
            var steps       = destinyPath.split('/')
            var actualPath  = Object.assign({}, startingPoint)
            for(var step of steps){
                if(Object.keys(actualPath.pathObj).includes(step)){
                    actualPath.pathHistory.push(Object.assign({}, actualPath.pathObj));
                    actualPath.pathObj  = actualPath.pathObj[step]
                    actualPath.text     += step + '/'
                    deepness            += 1
                }
            }
            if(deepness != steps.length){ this.emitMessage('warning', `Path not found!`) }
            return (deepness == steps.length) ? actualPath : this.currentPath 
        },

        // Transforma entrada no usuário em função
        executeCommand(){
            var treatedInput        = this.inputConsole.trim()          // Removendo espaços do começo e do fim da entrada
            var splitInput          = treatedInput.split(' ');          // Separando por ' '
            var commandName         = splitInput[0];                    // 'comandoNome' é a primeira parte
            splitInput.shift();                                         // remove a primeira parte
            var commandArgs         = splitInput;                       // 'comandoArgumentos' é o resto da lista
            var commandObj          = this.validCommands[commandName];  // 'comandoObjeto' = 'comandosVálidos[comandoNome]' virá undefined se não existir
            this.emitMessage('normal', this.inputConsole);              // Emite o comando inserido
            // Se não existir emite mensagem e retorna
            if(!commandObj) { this.emitMessage('warning', `Command '${commandName}' not found!`); return; }
            commandObj.function(commandArgs);
        },

        // Console commands
        commandEcho(args){
            var text = Array.isArray(args) ? args[0] : args
            this.emitMessage('normal', '"'+text+'"')
        },

        commandClear(){
            this.history = [];
        },

        commandList(args){
            var diretoryToList = args[0] ? this.isPathValid(args[0]).pathObj : this.currentPath.pathObj ;
            this.history.push({
                path: this.currentPath.text,
                type: 'list',
                value: Object.keys(diretoryToList)
            });
        },

        commandMakeDiretory(args, startingPoint = this.currentPath.pathObj){
            // Trata o nome do diretório
            var diretoryName = Array.isArray(args) ? args[0] : args;
            // Se o nome do diretório conter '.' é inválido [return]
            if(diretoryName.includes('.')){ this.emitMessage('warning', `'${diretoryName}' is not a valid name`); return; }
            // Se o nome do diretório não incluir '/' cria o diretório na localização atual [return]
            if(!diretoryName.includes('/')) { startingPoint[diretoryName] = {}; return;}
            // Separa o nome do diretório em '/'
            var directorySplit = diretoryName.split('/')
            // Se a primeira pasta do nome dividido não existir, cria
            if(!startingPoint[directorySplit[0]]){ startingPoint[directorySplit[0]] = {} }
            // Recupera o diretório criado
            var createdDir = startingPoint[directorySplit[0]];
            // Remove-o da lista
            directorySplit.shift()
            // Cria os diretórios restantes
            this.commandMakeDiretory(directorySplit.join('/'), createdDir)
        },

        commandMoveTo(args){
            // Se chamar o comando com 0 ou mais de 1 argumento
            if(args.length > 1 || args.length == 0){ this.emitMessage('warning', `Wrong parameters structure, use 'cd folder/folder/'`); return; }
            // Trata o nome do diretório
            var path = Array.isArray(args)?args[0]:args;
            // Se o nome não for '..' defina o caminho atual como this.isPathValid(path) [return]
            if(path  != '..') { this.currentPath = this.isPathValid(path); return; }
            // Se o nome for '..' e já estiver no topo emite mensagem de erro [return]
            if(path == '..' && this.currentPath.pathHistory.length <= 0) { this.emitMessage('warning', 'Cannot get back one level'); return; }
            // Define o último pathHistory como o path atual
            this.currentPath.pathObj = this.currentPath.pathHistory[this.currentPath.pathHistory.length-1]
            // Remove o último pathHistory da lista
            this.currentPath.pathHistory.splice(-1, 1)
            // Corrige o texto do currentPath.text
            var splitPathText = this.currentPath.text.split('/')
            if(splitPathText.length > 1){ splitPathText.pop(); splitPathText.pop(); }
            this.currentPath.text = splitPathText.join('/') + '/'
        },

        commandTouch(args, startingPoint=this.currentPath){
            // Trata os argumentos
            var userInput       = Array.isArray(args)?args[0]:args;
            var fileContent     = args[1]  || "Insert text here..."
            // Quebra o texto nas '/'
            var fileSplitText   = userInput.split('/');
            // Remove o primeiro item do texto quebrado
            var fileNameText    = fileSplitText.pop();
            // Redefine o pathObj do arquivo se não for o atual
            var filePathObj     = fileSplitText.length > 0 ? (this.isPathValid(fileSplitText.join('/'), startingPoint)) : (startingPoint);
            // Se não encontrar o pathObj, emite mensagem [return]
            if(!filePathObj){ this.emitMessage('warning', `Directory '${fileSplitText.join('/')}' not found`); return; }
            // Se o arquivo não tiver o formato especificado, emite mensagem [return]
            if(!fileNameText.includes('.')) { this.emitMessage('warning', `Missing filename`); return; }
            // Cria o arquivo / redefine o texto do arquivo
            filePathObj.pathObj[fileNameText] = fileContent;
        },

        commandCat(args, startingPoint=this.currentPath){
            // Trata os argumentos
            var userInput       = Array.isArray(args)?args[0]:args;
            // Quebra o texto nas '/'
            var fileSplitText   = userInput.split('/');
            // Remove o primeiro item do texto quebrado
            var fileNameText    = fileSplitText.pop();
            // Redefine o pathObj do arquivo se não for o atual
            var filePathObj     = fileSplitText.length > 0 ? (this.isPathValid(fileSplitText.join('/'), startingPoint)) : (startingPoint);
            // Se o arquivo não existir, emite mensagem [return]
            if(!filePathObj){ this.emitMessage('warning', `Directory '${fileSplitText.join('/')}' not found`); return; }
            // Emite comando Echo com o conteúdo do arquivo
            this.commandEcho(filePathObj.pathObj[fileNameText]);
        }
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