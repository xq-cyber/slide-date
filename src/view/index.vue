<template>
  <div>
    <!-- <SlideDate @getDate="getDate"></SlideDate> -->
    <SlideDate height="210" itemHight="30"  @getDate="getDate"></SlideDate>
  </div>
</template>

<script>
import SlideDate from "@/components/slide-date"
export default {
  name: 'Index',
  components:{SlideDate},
  data () {
    return {
      
    }
  },
  methods:{
    getDate(emitVal){
      console.log(emitVal)
    },
  }
}
</script>

<style scoped>

</style>
