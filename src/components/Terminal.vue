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
                        <b class="text-primary">{{currentPathText}}</b>$
                        <input type="text" v-model="inputConsole" ref="DOMinputTerminal" class="terminal-input bg-dark text-white border-0 w-75" placeholder="Use the command 'help' to see all commands">
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
                    <div class="col border rounded m-1 pt-2 bg-white">
                        <h5>
                            <small class="text-muted">
                                <i class="fa fa-battery-half">&nbsp;</i>
                                Allocated
                            </small>
                        </h5>
                        <div class="progress">
                            <div class="progress-bar" role="progressbar" :style="`width:${((memory.blocks.length/32000)*100).toFixed(2)}%;`"></div>
                        </div>
                        <h5>{{((memory.blocks.length/32000)*100).toFixed(2)}}<small class="text-muted">%</small></h5>
                        <h5>{{memory.blocks.length}}<small class="text-muted">&nbsp;blocks</small></h5>
                        <h5>{{memory.blocks.length*4/1000}}<small class="text-muted">&nbsp;Mb / 128 Mb</small></h5>
                    </div>
                    <div class="col border rounded m-1 pt-2 bg-white">
                        <h5>
                            <small class="text-muted">
                                <i class="fa fa-battery-empty">&nbsp;</i>
                                Empty
                            </small>
                        </h5>
                        <div class="progress">
                            <div class="progress-bar bg-info" role="progressbar" :style="`width:${((memory.empties.length/memory.blocks.length)*100).toFixed(2)}%;`"></div>
                        </div>
                        <h5>{{((memory.empties.length/memory.blocks.length)*100).toFixed(2)}}<small class="text-muted">%</small></h5>
                        <h5>{{memory.empties.length}}<small class="text-muted">&nbsp;blocks</small></h5>
                        <h5>{{memory.empties.length*4/1000}}<small class="text-muted">&nbsp;Mb / 128 Mb</small></h5>
                    </div>
                </div>
                <div class="row mb-5">
                    <div class="col-12 col-md-6 col-lg-4 px-3 py-2" v-for="(block, index) in memory.nodes.filter((v)=> {return v;})" :key="'block_'+index">
                        <div class="row border rounded p-2 bg-white">
                            <div class="col-12">
                                <h6>
                                    {{block.label}}
                                    <i :class="'float-end fa fa-'+(block.type=='diretory'?'folder':'file')"></i>
                                </h6>
                            </div>
                            <div class="col-4">
                                <h6>{{((block.blocks.length/memory.blocks.length)*100).toFixed(2)}}<small class="text-muted">%</small></h6>
                            </div>
                            <div class="col-4">
                                <h6>{{block.blocks.length}}<small class="text-muted">&nbsp;block{{block.blocks.length>1?'s':''}}</small></h6>
                            </div>
                            <div class="col-4">
                                <h6>{{block.blocks.length*4/1000}}<small class="text-muted">&nbsp;/128 Mb</small></h6>
                            </div>
                        </div>
                    </div>
                </div>
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
            memory: {
                nodes:      [],
                blocks:     [],
                empties:    []
            },
            
            currentNodeIndex: 0,
            currentPathText:  "",

            blockColors: ["54478c","2c699a","048ba8","0db39e","16db93","83e377","b9e769","efea5a","f1c453","f29e4c"],

            history: [],

            inputConsole: "",
            inputHistory: [],
            inputHistoryPosition: 0,

            validCommands: {
                "help": {
                    description:    "Show the help content into the terminal",
                    function:       this.commandHelp
                },
                "echo": {
                    description:    "Write content into file"
                    + "<br>Input<br>"
                    + '<i class="p-1 rounded bg-dark text-white"><b>echo "</b> content <b>" >> file.txt</b> </i><br>'
                    + "Output<br>"
                    + "<i>Insert content into file <b>file.txt</b></i>",
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
                    description:    "List all the paths and files in the diretory"
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
                    function:       this.commandLs
                },
                "mkdir": {
                    description:    "Create a new diretory"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>mkdir</b> dirName</i><br>"
                    + "Output<br>"
                    + "<i>Creates diretory 'dirName/' inside current path</i><br>"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>mkdir</b> home/dirName</i><br>"
                    + "Output<br>"
                    + "<i>Creates diretory 'dirName/' inside path 'home/'/</i>",
                    function:       this.commandMkdir
                },
                "cd":{
                    description:    "Move to the diretory"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>cd</b> home</i><br>"
                    + "Output<br>"
                    + "<i>Sets current path as 'home/'/</i>",
                    function:       this.commandCd
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
                    function:       this.commandRm
                },
                "rmdir":{
                    description:    "Remove a diretory"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>rmdir</b> home/pictures</i><br>"
                    + "Output<br>"
                    + "<i>Removes diretory 'home/picutres/' if its empty</i>",
                    function:       this.commandRmdir
                },
                "mv":{
                    description:    "Rename a file or folder"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>mv</b> home/pictures home/images</i><br>"
                    + "Output<br>"
                    + "<i>Renames diretory 'home/picutres/' to 'home/images'</i>",
                    function:       this.commandMv
                },
                "cp":{
                    description:    "Clone a file"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>cp</b> home/documents/README.txt home/pictures/CLONE.txt</i><br>"
                    + "Output<br>"
                    + "<i>Clones file 'home/documents/README.txt' into 'home/pictures/CLONE.txt'</i>",
                    function:       this.commandCp
                },
                "save":{
                    description:    "Save current state"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>save</b></i><br>"
                    + "Output<br>"
                    + "<i>Save current filesystem state</i>",
                    function:       this.commandSave
                },
                "reset":{
                    description:    "Reset the saved state to default initial state"
                    + "<br>Input<br>"
                    + "<i class='p-1 rounded bg-dark text-white'><b>reset</b></i><br>"
                    + "Output<br>"
                    + "<i>Memory state and current state is reseted to default</i>",
                    function:       this.commandReset
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
        this.commandLoad();
        this.updatePathText();
        this.$refs.DOMdivContainer.addEventListener('click', () => { this.$refs.DOMinputTerminal.focus() });
    },

    watch: {
        history: {
            deep: true,
            handler: function() {
                var elem = this.$refs.DOMdivHistory
                setTimeout(function(){elem.scrollTop = elem.scrollHeight;}, 50);
            }
        },
        memory: {
            deep: true,
            handler: function() {
                console.log(this.memory.blocks)
            }
        }
    },

    methods: {
        // Transforma entrada no usuário em função
        executeCommand(){
            var treatedInput        = this.inputConsole.trim()                  // Removendo espaços do começo e do fim da entrada
            var splitInput          = treatedInput.split(' ');                  // Separando por ' '
            var commandName         = splitInput.shift();                       // 'comandoNome' é a primeira parte
            var commandArgs         = splitInput;                               // 'comandoArgumentos' é o resto da lista
            var commandObj          = this.validCommands[commandName];          // 'comandoObjeto' = 'comandosVálidos[comandoNome]' virá undefined se não existir
            this.consolePrintMessage('normal', this.inputConsole);              // Emite o comando inserido
            this.inputHistory.push(this.inputConsole);
            this.inputHistoryPosition = this.inputHistory.length;
            // Se não existir emite mensagem e retorna
            if(!commandObj) { this.consolePrintMessage('warning', `Command '${commandName}' not found!`, 'Error'); return; }
            commandObj.function(commandArgs);
        },

        updatePathText(){
            var path = [];
            var temp = this.memory.nodes[this.currentNodeIndex];
            while(temp.parent != null){
                path.push(temp.label);
                temp = this.memory.nodes[temp.parent]
            }
            path.reverse()
            this.currentPathText = '~/'+path.join('/')+(path.length?'/':'');
        },

        generateHexColor(id){
            var colorId = id.toString()[0];
            return '#'+this.blockColors[colorId]
        },
        
        consolePrintMessage(type, content, title=null){
            var text = (title ? (type != 'normal' ? `<b class="text-${type}">${title} - </b>` : `<b>${title} - </b>`) : '') + `${content}`
            this.history.push({ path: this.currentPathText, type:'unique', value:text });
        },

        consolePrintList(content){
            this.history.push({ path:this.currentPathText, type:'list', value:content });
        },

        consolePrintHelp(content){
            this.history.push({ path:this.currentPathText, type:'help', value:content });
        },

        allocateBlocks(nodeId, blockQuantity){
            var blocks = [];
            while(blockQuantity--){
                var id = (this.memory.empties.length > 0) ? this.memory.empties.shift() : this.memory.blocks.length;
                this.memory.blocks[id] = nodeId;
                blocks.push(id);
            }
            return [...blocks];
        },

        createDiretory(label, parent, children){
            var id = this.memory.nodes.length;
            this.memory.nodes.push({id: id,  type:'folder', label:label, parent: parent, children: children, blocks: this.allocateBlocks(id, 1)});
            if(this.memory.nodes[parent]) { this.memory.nodes[parent].children.push(id); }
            return id;
        },

        createFile(label, parent, content){
            var id = this.memory.nodes.length;
            this.memory.nodes.push({id: id, type:'file', label:label, parent: parent, content: content, blocks: this.allocateBlocks(id, Math.ceil(content.length/4))});
            if(this.memory.nodes[parent]) { this.memory.nodes[parent].children.push(id); }
            return id;
        },

        deleteNode(id){
            var fileToDelete = this.memory.nodes[id];
            console.log(fileToDelete)
            for(var block of fileToDelete.blocks){
                this.memory.blocks[block] = null;
                this.memory.empties.push(block);
            }
            var parent = this.memory.nodes[fileToDelete.parent];
            var parentChildrenIndex = parent.children.indexOf(id);
            parent.children.splice(parentChildrenIndex, 1);
            this.memory.nodes[id] = null;
        },


        // Console commands
        commandHelp(){
            var content = [];
            for(var command of Object.keys(this.validCommands)){
                content.push(command);
                content.push(this.validCommands[command].description);
            }
            this.consolePrintHelp(content);
        },

        commandSave(){
            var memoryJSON = JSON.stringify(this.memory)
            localStorage.setItem('memory', memoryJSON);
        },

        commandReset(){
            localStorage.setItem('memory', null);
            this.commandLoad();
        },

        commandLoad(){
            var memoryJSON = localStorage.getItem('memory');
            // If found memory to load - load the last state
            var memory = memoryJSON ? JSON.parse(memoryJSON) : null;
            if(memory){ this.memory = JSON.parse(memoryJSON); return; }
            console.log('Creating default memory state...');
            // Else create default memory state
            this.memory = { nodes:[], blocks: [], empties: [] };
            this.createDiretory('~',            null,   []);
            this.createDiretory('home',         0,      []);
            this.createDiretory('pictures',     1,      []);
            this.createDiretory('documents',    1,      []);
            this.createDiretory('usr',          0,      []);
            this.createDiretory('libs',         4,      []);
            this.createDiretory('bin',          4,      []);
            this.createFile('README.txt',       3,      'Welcome to the terminal!');
            this.createFile('helloWorld.txt',   3,      '<b>Hello world!</b>');
            console.log(this.memory.nodes.map((v)=> {return v.label + '- ['+v.children+']';} ));
        },

        commandClear(){
            this.history = [];
        },
        
        getPathIndex(path, parentId = this.currentNodeIndex){
            var deepness  = 0;
            var steps     = path.split('/');
            var tempId    = parentId;
            if(steps[0] == '~'){
                steps.shift();
                tempId = 0;
            }
            if(steps[0] == '.'){ steps.shif(); }
            for(var nodeLabel of steps){
                var node = this.memory.nodes[tempId]
                if(nodeLabel == '..' && 'parent' in node){
                    deepness += 1;
                    tempId = node.parent;
                }else{
                    for(var child of node.children){
                        var childNode = this.memory.nodes[child]
                        if(childNode.label == nodeLabel){
                            deepness += 1;
                            tempId = childNode.id; 
                            break;
                        }
                    }
                }
            }
            return (deepness == steps.length) ? tempId : null;
        },
        
        commandEcho(args, parentId = this.currentNodeIndex){
            if(args.length < 3) { this.consolePrintMessage('danger', 'Missing parameters', 'Error'); return;}
            
            var path = args.pop() //a.txt
            args.pop()
            var content = args.join(" ")
            content.replace('"',null);
           

            var fileId = this.getPathIndex(path, parentId);
            if(fileId == null) { this.consolePrintMessage('danger', 'File not found', 'Error'); return; }
            if(!this.memory.nodes[fileId].label.includes('.')) { this.consolePrintMessage('danger', 'Cant insert content into diretory', 'Error'); return; }

            var oldMemory = this.memory.nodes[fileId].blocks.length;
            this.memory.nodes[fileId].content += content;
            var newMemory = Math.ceil(this.memory.nodes[fileId].content.length/4);
            
            var blocskDiff = oldMemory- newMemory;
            if(blocskDiff > 0){
                this.memory.nodes[fileId].blocks.push(this.allocateBlocks(fileId, blocskDiff))
            }else{
                blocskDiff *= -1
                while(blocskDiff--){
                    this.memory.blocks[this.memory.nodes[fileId].blocks.pop()] = null;
                }
            }
        },

        commandLs(args, parentId = this.currentNodeIndex){
            // Definindo qual o nodo a ter os filhos listados
            var nodeId      = args.length ? this.getPathIndex(args[0], parentId) : parentId;
            var node        = this.memory.nodes[nodeId];
            // Tratamento de erros
            if(!node)                       { this.consolePrintMessage('danger', 'diretory not found', 'Error');       return; }
            if(node.label.includes('.'))    { this.consolePrintMessage('danger', `Cant list '${node.label}'`, 'Error'); return; }
            if(node.children.length == 0)   { this.consolePrintList(['Empty folder']);                                  return; }
            // Printando label dos filhos
            this.consolePrintList(node.children.map( (n) => { return this.memory.nodes[n].label; }));
        },

    
        commandMkdir(args, parentId = this.currentNodeIndex){
            // Definindo qual o nodo que o usuario quer entrar
            var dirName = args.length ? args[0] : null;
            // Se não inserir o nome do diretório
            if(dirName.includes('.')) { this.consolePrintMessage('danger', 'diretory names must NOT include special symbols', 'Error'); return; }
            if(dirName == null) { this.consolePrintMessage('danger', 'Missing name of diretory', 'Error'); return; }
            // Se for um path encontra o id do pai onde vai ser criado
            if(dirName.includes('/')){
                var splitPath   = dirName.split('/');
                dirName         = splitPath.pop();
                parentId        = this.getPathIndex(splitPath.join('/'), parentId);
            }
            // Se tentar encontrar o id do pai mas não encontrar
            if(parentId == null) { this.consolePrintMessage('danger', 'diretory not found', 'Error'); return; }

            // Verifica se o pai não tem filho com mesmo nome
            var parentNode    = this.memory.nodes[parentId]
            var childLabels   = parentNode.children.map( (childId) => { return this.memory.nodes[childId].label });
            if(childLabels.includes(dirName)){ this.consolePrintMessage('danger', `diretory '${splitPath.join('/')}/${dirName}' already exists`, 'Error'); return;}
            
            // Cria o diretório
            this.createDiretory(dirName, parentId, []);
        },

        commandCd(args, parentId = this.currentNodeIndex){
            // Definindo qual o nodo que o usuario quer entrar
            var dirName = args.length ? args[0] : null;
            // Se não inserir o nome do diretório
            if(dirName == null) { this.consolePrintMessage('danger', 'Missing name of diretory', 'Error'); return; }
            var target = this.getPathIndex(dirName, parentId)
            if(target == null) { this.consolePrintMessage('danger', `Path '${dirName}' not found`, 'Error'); return; }
            this.currentNodeIndex = target;
            this.updatePathText();
        },

        commandTouch(args, parentId=this.currentNodeIndex){
            // Definindo qual o nodo que o usuario quer entrar
            var fileName    = args.length ?  args.shift()    : null;
            var fileContent = args.length ?  args.join(' ')  : ''
            // Se não inserir o nome do diretório
            if(fileName == null) { this.consolePrintMessage('danger', 'Missing name of file', 'Error'); return; }
            if(!fileName.includes('.')) { this.consolePrintMessage('danger', 'Missing file format', 'Error'); return; }
            // Se for um path encontra o id do pai onde vai ser criado
            if(fileName.includes('/')){
                var splitPath   = fileName.split('/');
                fileName        = splitPath.pop();
                parentId        = this.getPathIndex(splitPath.join('/'), parentId);
            }
            // Se tentar encontrar o id do pai mas não encontrar
            if(parentId == null) { this.consolePrintMessage('danger', 'diretory not found', 'Error'); return; }

            // Verifica se o pai não tem filho com mesmo nome
            var parentNode    = this.memory.nodes[parentId]
            var childLabels   = parentNode.children.map( (childId) => { return this.memory.nodes[childId].label });
            if(childLabels.includes(fileName)){ this.consolePrintMessage('danger', `File '${fileName}' already exists`, 'Error'); return;}
            
            // Cria o arquivo
            this.createFile(fileName, parentId, fileContent);
        },

        commandCat(args, parentId=this.currentPath){
            var path    = args.length ?  args[0]    : null;
            var fileId  = this.getPathIndex(path, parentId);
            if(fileId == null) { this.consolePrintMessage('danger', 'File not found', 'Error'); return;}
            this.consolePrintMessage('normal', this.memory.nodes[fileId].content, `'${this.memory.nodes[fileId].label}'`)
        },

        commandRm(args, parentId=this.currentPath){
            var path    = args.length ?  args[0]    : null;
            var fileId  = this.getPathIndex(path, parentId);
            if(fileId == null) { this.consolePrintMessage('danger', 'File not found', 'Error'); return;}
            if(this.memory.nodes[fileId].type=='diretory') { this.consolePrintMessage('danger', "Use 'rmdir' to remove diretory", 'Error'); return;}
            this.deleteNode(fileId);
        },

        commandRmdir(args, parentId=this.currentPath){
            var path    = args.length ?  args[0]    : null;
            var diretoryId  = this.getPathIndex(path, parentId);
            if(diretoryId == null) { this.consolePrintMessage('danger', 'Diretory not found', 'Error'); return;}
            if(this.memory.nodes[diretoryId].type=='file') { this.consolePrintMessage('danger', "Use 'rm' to remove a file", 'Error'); return;}
            if(this.memory.nodes[diretoryId].children.length) { this.consolePrintMessage('danger', "Diretory is not empty", 'Error'); return;}
            this.deleteNode(diretoryId);
        },

        commandMv(args, parentId=this.currentPath){
            var path    = args.length ?     args[0]     : null;
            var newName = args.length ?     args[1]     : null;
            var nodeId  = this.getPathIndex(path, parentId);
            if(nodeId == null) { this.consolePrintMessage('danger', 'Path not found', 'Error'); return;}
            if(!newName) { this.consolePrintMessage('danger', 'Missing new name', 'Error'); return;}
            if(this.memory.nodes[nodeId].type=='file' && !newName.includes('.')) { this.consolePrintMessage('danger', 'Missing file format', 'Error'); return;}
            if(!this.memory.nodes[nodeId].type=='file' && newName.includes('.')) { this.consolePrintMessage('danger', 'diretory names must NOT include special symbols', 'Error'); return; }
            this.memory.nodes[nodeId].label = newName;
        },

        commandCp(args, parentId=this.currentPath){
            var pathOriginal    = args.length ?     args[0]     : null;
            var pathClone       = args.length ?     args[1]     : null;
            if(pathClone == null) { this.consolePrintMessage('danger', 'Missing clone path', 'Error'); return;}

            var originalId = this.getPathIndex(pathOriginal, parentId);
            if(originalId == null) { this.consolePrintMessage('danger', 'Original file not found', 'Error'); return;}

            var originalNode = this.memory.nodes[originalId];
            this.commandTouch([pathClone, originalNode.content]);
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