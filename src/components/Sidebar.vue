<template>
<div class="border">
    <div class="p-2" v-if="models.length != 0">
        <p>Object list</p>
        <hr/>
        <ul class="list-unstyled" v-for="model in models">
            <li>
                <a href="#">{{model.name}}</a>
            </li>
        </ul>

    </div>
    <div v-else>
        <p class="p-2">No models added.</p>
    </div>
</div>
</template>

<script>
import { ref, onMounted, onUnmounted } from 'vue'

export default {
    name: 'Sidebar',
    props: {
        models: {
            type: Array,
            required: true
        }
    },
    setup() {

    }
}


</script>