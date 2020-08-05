# input限制只能输入正整数

```
  // 只能输入正整数
  <el-input 
    @input="(val)=>{textVal = val.replace(/[^\d]/g, '')}" 
    placeholder="退出重置（分钟）" 
    v-model="textVal">
  </el-input>
  // 只能输入正整数和：
  <el-input 
    @input="(val)=>{textVal = val.replace(/[^\d:]/g, '')}" 
    placeholder="退出重置（分钟）" 
    v-model="textVal">
  </el-input>
```

