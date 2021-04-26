<template>
    <div class="modal fade" id="modalHelp" tabindex="-1" aria-labelledby="modalHelpLabel" aria-hidden="true">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="modalHelpLabel">
                        <i class="fa fa-question-circle">&nbsp;</i>
                        Help
                    </h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body text-start">
                    <ul class="list-group">
                        <li class="list-group-item" v-for="(command, commandName) in commands" :key="'valid_command_'+commandName">
                            <h6 data-bs-toggle="collapse" :href="'#commands_help_collapse_'+commandName">> {{commandName}}</h6>
                            <div class="collapse pb-2" :id="'commands_help_collapse_'+commandName">
                                <span class="text-muted" v-html="command.description"></span>
                            </div>
                    
                        </li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'ModalHelp',

    props: {
        commands: Object
    }
}
</script>