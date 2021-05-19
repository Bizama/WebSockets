<template>
    <div class="information-box">
        <div>
            <span>{{ flight_list['code'] }}</span>
        </div> 
        <div>
            <span>{{ flight_list['airline'] }}</span>
        </div> 
         <div>
            <span>{{ flight_list['plane'] }}</span>
        </div> 
        <div>
            <span>{{ flight_list['passengers']}}</span>
        </div>
    </div>
</template>
<script>

export default ({
    props: {
        flight_list: Object
    },
    data() {
        return {
            hello_world: 'Hello World', 
        }
    }
})
</script>
<style>
.information-box {
    border-color: hotpink;
    height: 100px;
    width: 100%;
    display: flex;
    flex-direction: column;
    margin-right: 32px;
    padding-right: 32px;
}
</style>