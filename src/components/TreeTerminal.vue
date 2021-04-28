<template>
    <div class="container">
        <div class="row">
            <div class="col-12">
                <h4 class="mt-3 text-start">
                    <i class="fa fa-terminal">&nbsp;</i>
                    Terminal
                    <a data-bs-toggle="modal" data-bs-target="#modalHelp" class="link-secondary fa fa-question-circle float-end text-decoration-none"></a>
                </h4>
                <div class="terminal p-3 bg-dark text-white position-relative text-start rounded text-break" ref="DOMdivContainer">
                    <div class="terminal-history mb-3" ref="DOMdivHistory" style="overflow-y:auto">
                        <div v-for="(h, mainIndex) in history" :key="'history_'+mainIndex" class="container">
                            <TerminalHistory :history="h"></TerminalHistory>
                        </div>
                    </div>
                    <div class="terminal-entry position-absolute bottom-0 left-0 pb-1 w-100">
                        <b class="text-primary">{{currentPath.text}}</b>$
                        <input type="text" v-model="inputConsole" ref="DOMinputTerminal" class="terminal-input bg-dark text-white border-0 w-75">
                    </div>
                </div>
            </div>
        </div>
        <div class="row text-start">
            <div class="col-12">
                <h4 class="mt-3">
                    <i class="fa fa-database">&nbsp;</i>
                    Memory
                </h4>
                <div class="row">
                    <div class="col border rounded m-1 pt-2">
                        <h5>
                            <small class="text-muted">
                                <i class="fa fa-battery-half">&nbsp;</i>
                                Allocated
                            </small>
                        </h5>
                        <h5>{{((memory.blocks.length/32000)*100).toFixed(2)}}<small class="text-muted">%</small></h5>
                        <h5>{{memory.blocks.length}}<small class="text-muted">&nbsp;blocks</small></h5>
                        <h5>{{memory.blocks.length*4/1000}}<small class="text-muted">&nbsp;Mb / 128 Mb</small></h5>
                    </div>
                    <div class="col border rounded m-1 pt-2">
                        <h5>
                            <small class="text-muted">
                                <i class="fa fa-battery-empty">&nbsp;</i>
                                Empty
                            </small>
                        </h5>
                        <h5>{{((memory.empties.length/memory.blocks.length)*100).toFixed(2)}}<small class="text-muted">%</small></h5>
                        <h5>{{memory.empties.length}}<small class="text-muted">&nbsp;blocks</small></h5>
                        <h5>{{memory.empties.length*4/1000}}<small class="text-muted">&nbsp;Mb / 128 Mb</small></h5>
                    </div>
                </div>
                <div class="row">
                    <div class="col-12 col-md-6 col-lg-4 px-3 py-2" v-for="(block, index) in blocksToRender" :key="'block_'+index">
                        <div class="row border rounded p-2">
                            <div class="col-12">
                                <h6>
                                    Node #{{index}}
                                    <i class="float-end fa fa-folder"></i>
                                </h6>
                            </div>
                            <div class="col-4">
                                <h6>{{((block/memory.blocks.length)*100).toFixed(2)}}<small class="text-muted">%</small></h6>
                            </div>
                            <div class="col-4">
                                <h6>{{block}}<small class="text-muted">&nbsp;blocks</small></h6>
                            </div>
                            <div class="col-4">
                                <h6>{{block*4/1000}}<small class="text-muted">&nbsp;/128 Mb</small></h6>
                            </div>
                        </div>
                    </div>
                </div>
                <!--
                <div class="row mb-5 p-3 rounded bg-dark ">
                    <div class="col-12 row gx-1 gy-1 container overflow-auto" style="max-height:300px;">
                        <div 
                        class="col-1" 
                        v-for="(block, index) in memory.blocks" 
                        :key="'memory_block_'+index" 
                        :ref="'memory_block_'+index">
                            <div class="p-1 rounded" :style="'background-color:'+(block != null ? generateHexColor(block) : '#eee')"></div>
                        </div>
                    </div>
                    <div class="progress bg-white col-12 px-0 m-1">
                        <div 
                        v-for="(block, index) in memory.blocks" 
                        :key="'memory_block_'+index" 
                        :ref="'memory_block_'+index"
                        class="progress-bar border border-white rounded" 
                        role="progressbar" 
                        data-bs-toggle="tooltip" data-bs-placement="top" :title="'Node #'+block"
                        :style="'width:' + 100/memory.blocks.length + '%;' + 'background-color:'+(block != null ? generateHexColor(block) : '#eee')">
                        </div>
                    </div>
                </div>-->
            </div>
        </div> 
        <ModalHelp :commands="validCommands"></ModalHelp>
    </div>
</template>

<script>
import TerminalHistory from './TerminalHistory'
import ModalHelp from './ModalHelp'

export default {
    name: 'Terminal',

    components: {
      TerminalHistory,
      ModalHelp
    },

    data() {
        return {

            /*
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
            },*/

            absolutePath: {
                memoryId:       -1,
                memoryBlocks:   [],
                nodes:{
                    "home":{
                        memoryId:       0,
                        memoryBlocks: [0],
                        nodes:          {
                            "README.txt": {
                                memoryId:       2,
                                memoryBlocks:   [2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33],
                                nodes:          "Welcome to the console simulator!"
                            }
                        }
                    },
                    "usr": {
                        memoryId:       1,
                        memoryBlocks: [0],
                        nodes:          {}
                    }
                }
            },

            memory: {
                blocks:     [0, 1, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2],
                empties:    [],
                currId:     2
            },

            blockColors: ["54478c","2c699a","048ba8","0db39e","16db93","83e377","b9e769","efea5a","f1c453","f29e4c"],
            blocksToRender: [],

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

            inputHistory: [],
            inputHistoryPosition: 0,

            validCommands: {
                "echo": {
                    description:    "Prints message"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>echo</b> message</i><br>"
                    + "Output<br>"
                    + "<i class='p-1 rounded bg-dark text-white'>~/ message</i>",
                    function:       this.commandEcho
                },
                "clear":{
                    description:    "Clears the console"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>clear</b></i><br>"
                    + "Output<br>"
                    + "<i>Empty console</i>",
                    function:       this.commandClear
                },
                "ls":{
                    description:    "List all the paths and files in the directory"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>ls</b></i><br>"
                    + "Output<br>"
                    + "<i class='p-1 rounded bg-dark text-info'>home     usr </i><br>"
                    + "<i>All files and folders inside current folder</i><br>"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>ls</b> home</i><br>"
                    + "Output<br>"
                    + "<i class='p-1 rounded bg-dark text-white'>README.txt</i><br>"
                    + "<i>All files and folders inside home/</i>",
                    function:       this.commandList
                },
                "mkdir": {
                    description:    "Create a new directory"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>mkdir</b> dirName</i><br>"
                    + "Output<br>"
                    + "<i>Creates directory 'dirName/' inside current path</i><br>"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>mkdir</b> home/dirName</i><br>"
                    + "Output<br>"
                    + "<i>Creates directory 'dirName/' inside path 'home/'/</i>",
                    function:       this.commandMakeDiretory
                },
                "cd":{
                    description:    "Move to the directory"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>cd</b> home</i><br>"
                    + "Output<br>"
                    + "<i>Sets current path as 'home/'/</i>",
                    function:       this.commandMoveTo
                },
                "touch":{
                    description:    "Create new file"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>touch</b> home/filename.txt File content goes here</i><br>"
                    + "Output<br>"
                    + "<i>Creates fileName.txt with content 'File content goes here' (Accepts HTML) inside 'home/'</i>",
                    function:       this.commandTouch
                },
                "cat":{
                    description:    "Print file content"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>cat</b> home/README.txt</i><br>"
                    + "Output<br>"
                    + "<i>Print the file 'home/README.txt' content in the terminal (Accepts HTML)</i>",
                    function:       this.commandCat
                },
                "rm":{
                    description:    "Remove a file"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>rm</b> home/README.txt</i><br>"
                    + "Output<br>"
                    + "<i>Removes file 'home/README.txt', cannot be undone</i>",
                    function:       this.commandRemoveFile
                },
                "rmdir":{
                    description:    "<i class='fa fa-cog'>&nbsp;</i>Under development",
                    function:       this.commandRemoveFolder
                },
                "save":{
                    description:    "Save current state"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>save</b></i><br>"
                    + "Output<br>"
                    + "<i>Save current filesystem state</i>",
                    function:       this.commandSave
                },
                "load":{
                    description:    "Load last saved state"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>load</b></i><br>"
                    + "Output<br>"
                    + "<i>Load last saved state</i>",
                    function:       this.commandLoad
                }
            }
        }
    },

    mounted() {
        this.currentPath.pathObj    = this.absolutePath;
        this.currentPath.text       = "~/";
        this.$refs.DOMinputTerminal.addEventListener("keydown", e => {
            switch(e.key){
                case 'Enter':
                    this.executeCommand();
                    this.inputConsole = "";
                break;
                case 'ArrowUp':
                    this.inputHistoryPosition -= this.inputHistoryPosition > 0 ? 1 : 0
                    this.inputConsole = this.inputHistory[this.inputHistoryPosition]
                break;
                case 'ArrowDown':
                    this.inputHistoryPosition += this.inputHistoryPosition < this.inputHistory.length ? 1 : 0
                    this.inputConsole = this.inputHistory[this.inputHistoryPosition]
                break;
            }
            
        });
        this.memory.currId = 3;
        this.$refs.DOMdivContainer.addEventListener('click', () => { this.$refs.DOMinputTerminal.focus() });
    },

    watch: {
        history: {
            deep: true,
            handler: function() {
                var elem = this.$refs.DOMdivHistory
                setTimeout(function(){elem.scrollTop = elem.scrollHeight;},50);
            }
        },
        memory: {
            deep: true,
            handler: function(newVal) {
                this.blocksToRender = [];
                for(var block of newVal.blocks){
                    if(this.blocksToRender[block]){ this.blocksToRender[block] += 1 }
                    else { this.blocksToRender[block] = 1;} 
                }
            }
        }
    },

    methods: {

        generateHexColor(id){
            var colorId = id.toString()[0];
            return '#'+this.blockColors[colorId]
        },

        createDiretory(directoryName, path = this.currentPath){
            //Se já existir o diretório retorna
            if(Object.keys(path).includes(directoryName)) { this.emitMessage('danger', `Directory with name '${directoryName}' already exists`); return; }

            // Adicionando na memória
            var memoryBlock = this.memory.empties.length >= 1 ? this.memory.empties.shift() : ( this.memory.blocks.length < 32000 ? this.memory.blocks.length : null );
            if(!memoryBlock) { this.emitMessage('danger', 'Could not create diretory, no memory avaliable'); return; }
            this.memory.blocks[memoryBlock] = this.memory.currId;


            // Adicionando ao objeto
            path.nodes[directoryName] = {
                memoryId:       this.memory.currId,
                memoryBlocks:   [memoryBlock],
                nodes:          {}
            }
            
            // Incrementa o id atual
            this.memory.currId += 1;
        },

        createFile(fileName, fileContent="Write something with 'echo'", path=this.currentPath){
            var fileSize = fileContent.length
            var blocks = []
            var usedMemory = 0
            // Definindo blocos para alocar
            while(fileSize > 0){
                if(this.memory.empties.length > 0) { blocks.push(this.memory.empties.shift()); }
                else{ blocks.push(this.memory.blocks.length + usedMemory); usedMemory += 1; }
                fileSize -= 1;
            }
            if(usedMemory + this.memory.blocks.length > 32000) { this.emitMessage('danger', 'Could not create file, no memory avaliable'); return;}
            // Alocando memória
            for(var block of blocks){
                this.memory.blocks[block] = this.memory.currId;
            }
            // Adicionando ao objeto
            path.nodes[fileName] = {
                memoryId:       this.memory.currId,
                memoryBlocks:   blocks,
                nodes:          fileContent
            };
            // Incrementa o id atual
            this.memory.currId += 1;
        },

        // Delete file
        removeFile(fileName, path=this.currentPath){
            var fileObj = path.nodes[fileName];
            console.log(this.memory.blocks);
            for(var block of fileObj.memoryBlocks){
                this.memory.blocks[block] = null;
                this.memory.empties.push(block);
            }
            console.log(this.memory.blocks);
            delete path.nodes[fileName];
        },

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
                if(Object.keys(actualPath.pathObj.nodes).includes(step)){
                    actualPath.pathHistory.push(Object.assign({}, actualPath.pathObj));
                    actualPath.pathObj  = actualPath.pathObj.nodes[step]
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
            this.inputHistory.push(this.inputConsole);
            this.inputHistoryPosition = this.inputHistory.length;
            // Se não existir emite mensagem e retorna
            if(!commandObj) { this.emitMessage('warning', `Command '${commandName}' not found!`); return; }
            commandObj.function(commandArgs);
        },

        // Console commands
        commandSave(){
            var currentPathJSON = JSON.stringify(this.currentPath)
            var memoryJSON = JSON.stringify(this.memory)

            console.log(memoryJSON)
            localStorage.setItem('memory', memoryJSON);
            localStorage.setItem('currentPath', currentPathJSON);
        },

        commandLoad(){
            var memoryJSON = localStorage.getItem('memory')
            var currentPathJSON = localStorage.getItem('currentPath')
            this.memory = (memoryJSON) ? JSON.parse(memoryJSON) : this.memory;
            this.currentPath = (currentPathJSON) ? JSON.parse(currentPathJSON) : this.currentPath;
        },
        
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
                value: Object.keys(diretoryToList.nodes)
            });
        },

        commandMakeDiretory(args, startingPoint = this.currentPath.pathObj){
            // Trata o nome do diretório
            var diretoryName = Array.isArray(args) ? args[0] : args;
            // Se o nome do diretório conter '.' é inválido [return]
            if(diretoryName.includes('.')){ this.emitMessage('warning', `'${diretoryName}' is not a valid name`); return; }
            // Se o nome do diretório não incluir '/' cria o diretório na localização atual [return]
            if(!diretoryName.includes('/')) { this.createDiretory(diretoryName, startingPoint); return;}
            // Separa o nome do diretório em '/'
            var directorySplit = diretoryName.split('/')
            // Se a primeira pasta do nome dividido não existir, cria
            if(!startingPoint[directorySplit[0]]){ this.createDiretory([directorySplit[0]], startingPoint); }
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
            var userInput       = Array.isArray(args)? args.shift() : args;
            var fileContent     = Array.isArray(args)? args.join(' ') : "Insert text here..."
            // Quebra o texto nas '/'
            var fileSplitText   = userInput.split('/');
            // Remove o ultimo item do texto quebrado
            var fileNameText    = fileSplitText.pop();



            // Redefine o pathObj do arquivo se não for o atual
            var filePathObj     = fileSplitText.length > 0 ? (this.isPathValid(fileSplitText.join('/'), startingPoint)) : (startingPoint);


            // Se não encontrar o pathObj, emite mensagem [return]
            if(!filePathObj){ this.emitMessage('warning', `Directory '${fileSplitText.join('/')}' not found`); return; }
            // Se o arquivo não tiver o formato especificado, emite mensagem [return]
            if(!fileNameText.includes('.')) { this.emitMessage('warning', `Missing file format`); return; }
            // Cria o arquivo / redefine o texto do arquivo
            this.createFile(fileNameText, fileContent, filePathObj.pathObj);
        },

        commandCat(args, startingPoint=this.currentPath){
            // Trata os argumentos
            var userInput       = Array.isArray(args)?args[0]:args;
            if(!userInput) { this.emitMessage('warning', 'Missing file name'); return; }
            // Quebra o texto nas '/'
            var fileSplitText   = userInput.split('/');
            // Remove o primeiro item do texto quebrado
            var fileName        = fileSplitText.pop();
            if(!fileName.includes('.')) { this.emitMessage('warning', `Missing file format`); return; }
            // Redefine o pathObj do arquivo se não for o atual
            var filePathObj     = fileSplitText.length > 0 ? (this.isPathValid(fileSplitText.join('/'), startingPoint)) : (startingPoint);
            // Se o diretório não existir, emite mensagem [return]
            if(!filePathObj){ this.emitMessage('warning', `Directory '${fileSplitText.join('/')}' not found`); return; }
            // Se o arquivo não existir, emite mensagem [return]
            if(!filePathObj.pathObj.nodes[fileName]){ this.emitMessage('warning', `File '${userInput}' not found`); return; }
            // Emite comando Echo com o conteúdo do arquivo
            this.commandEcho(filePathObj.pathObj.nodes[fileName].nodes);
        },

        commandRemoveFile(args, startingPoint=this.currentPath){
            // Trata os argumentos
            var userInput       = Array.isArray(args)?args[0]:args;
            if(!userInput) { this.emitMessage('warning', 'Missing filename'); return; }
            // Quebra o texto nas '/'
            var fileSplitText   = userInput.split('/');
            // Remove o primeiro item do texto quebrado
            var fileName        = fileSplitText.pop();
            if(!fileName.includes('.')) { this.emitMessage('warning', `Missing file format`); return; }
            // Redefine o pathObj do arquivo se não for o atual
            var filePathObj     = fileSplitText.length > 0 ? (this.isPathValid(fileSplitText.join('/'), startingPoint)) : (startingPoint);
            // Se o diretório não existir, emite mensagem [return]
            if(!filePathObj){ this.emitMessage('warning', `Directory '${fileSplitText.join('/')}' not found`); return; }
            // Se o arquivo não existir, emite mensagem [return]
            if(!filePathObj.pathObj.nodes[fileName]){ this.emitMessage('warning', `File '${userInput}' not found`); return; }
            // Emite comando Echo com o conteúdo do arquivo
            this.removeFile(fileName, filePathObj.pathObj);
            this.commandEcho(`File '${userInput}' removed`);
        }
    }
}
</script>

<style scoped>

/* width */
::-webkit-scrollbar {
  width: 10px;
}

/* Track */
::-webkit-scrollbar-track {
  background: transparent; 
}
 
/* Handle */
::-webkit-scrollbar-thumb {
  background: white;
  border-radius: 15px; 
}

/* Handle on hover */
::-webkit-scrollbar-thumb:hover {
  background: #ddd; 
}

.terminal-input:focus {
    border: none;
    outline: none;
}

.terminal-history{
    height: 300px;
}

</style>