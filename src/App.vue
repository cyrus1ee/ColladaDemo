<template>
  <div id="app">
    <DaeReader/>
  </div>
</template>

<script>
//最初的版本，样板
//import FuncCube from "./components/base-class/Cube_Function";


//官方的版本
//import collada from './components/collada/ColladaTestVersion1'


//Dae读取样板
import DaeReader from "./components/daeFile/DaeReader";
//测试用
//import TestReader from "./components/daeFile/TestRead";
//测试用
//import StlTransToDaeDemo from "@/components/daeFile/StlTransToDaeDemo";

//读取Stl
//import StlReader from "./components/StlFile/StlReader";


export default {
    name: 'App',
    components: {
        //FuncCube
        //collada

        DaeReader
        //TestReader
        //StlTransToDaeDemo

        //StlReader
    }
}
</script>

<style>
#app{
  height: 1000px;
  width:1000px;
}
</style>
