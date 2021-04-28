<template>
    <div>
        <div v-if="history.type == 'unique'">
            <span v-html="history.path+'&nbsp;'" class="text-primary"></span>
            <span v-html="history.value"></span>
        </div>
        <div v-else-if="history.type == 'list'" class="row">
            <span v-for="(item, index) in history.value" :key="'history_'+mainIndex+'_value_'+index" v-html="item" class="col-4" :class="item.includes('.') || item.includes('Empty')?'text-muted':'text-info'"></span>
        </div>
    </div>
</template>

<script>
export default {
    name: 'TerminalHistory',

    props: {
        mainIndex: String,
        history: Object
    }
}
</script>