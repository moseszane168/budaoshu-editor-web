<template>
    <div class="wrapper">
        <hr />
        <button @click="send">send</button>
    </div>
</template>
<script>
import Bus from '../../utils/bus';
export default {
    data(){
        return {
            message:'',
        };
    },
    methods:{
        send(){
            this.message = '这是传递到second中的数据！';
            Bus.$emit('msg',this.message);
        },
    },
};
</script>