<template>
        <div :class="classname" :style="{'width': itemwidth-2 + 'px', 'height': itemwidth-2 + 'px', 'margin': margin + 'px'} ">{{num}}
                </div>
        <!-- <div :class="classname">
        </div> -->
</template>
<script>
export default {
    name: 'blockItem',
    props: ['itemwidth', 'margin', 'classname', 'num'],
}
</script>
<style scoped>
</style>