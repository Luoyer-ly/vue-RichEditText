<template>
  <div id="app">
    <el-row>
      <el-col :span="4">&nbsp;</el-col>
      <el-col :span="16">
        <quill-editor v-model="content"
                      ref="myQuillEditor"
                      :options="editorOption">
        </quill-editor>

        <br><br>
        <el-button type="primary" @click="showContent">输出JSON到控制台</el-button>
      </el-col>
      <el-col :span="4">&nbsp;</el-col>
    </el-row>
  </div>
</template>

<script>
  import Quill from 'quill'

  export default {
    data () {
      return {
        content: '<h2>I am Example</h2>',
        editorOption: {
          // some quill options
        }
      }
    },
    // manually control the data synchronization
    // 如果需要手动控制数据同步，父组件需要显式地处理changed事件
    methods: {
      onEditorBlur(quill) {
        console.log('editor blur!', quill)
      },
      onEditorFocus(quill) {
        console.log('editor focus!', quill)
      },
      onEditorReady(quill) {
        console.log('editor ready!', quill)
      },
      onEditorChange({ quill, html, text }) {
        console.log('editor change!', quill, html, text)
        this.content = html
      },
      showContent: function () {
//        console.log(this.editor.getContents());
        //获得内容对象Delta
        var content = this.editor.getContents().ops;
        var data = {
          blocks: [],
          entityMap: {}
        };
        var len=0; //data数组的长度
        for (let i = 0; i < content.length; i++) {
          if (content[i].insert.image) {
            //先组成对象数组
            data.blocks.push({
              depth: "0",
              entityRanges: [],
              inlineStyleRanges: [],
              text: "",
              type: "atomic"
            });
            var name = content[i].insert.image.substring(28, 50);
            var obj = {};
            obj[name] = {
              materialId: 'undefined',//未确定
              name: name,
              url: 'undefined',//未确定
              mutability: 'MUTABLE',
              type: 'SIDERIMAGE'
            };
            data.entityMap[name] = obj;
//            data.blocks[len].type = "atomic";
            let enR = {
              key: name,
              length: 1
            };
            data.blocks[len].entityRanges.push(enR);
            len++;
          }
          else {
            if(/^(\n)$/.test(content[i].insert.toString())) {
              if(len>0 && content[i].attributes && (data.blocks[len-1].text || data.blocks[len-1].type==='atomic')) {
                let style = Object.keys(content[i].attributes)[0];
                style += content[i].attributes[style];
                data.blocks[len - 1].type = (data.blocks[len - 1].type==='atomic'?'atomic':style);
                if(len>1 && data.blocks[len-2].text) {
                    data.blocks[len-2].type = (data.blocks[len-2].type==='atomic'?'atomic':style);
                }
                continue;
              }
              else {
                data.blocks.push({
                  depth: "0",
                  entityRanges: [],
                  inlineStyleRanges: [],
                  text: "",
                  type: "unstyled"
                });
                len++;
              }
            }
            else if(/^(\n)+$/.test(content[i].insert.toString())) {
               if(len>0) {
                   let style = Object.keys(content[i].attributes)[0];
                   style += content[i].attributes[style];
                   data.blocks[len-1].type = (data.blocks[len-1].type==='atomic'?'atomic':style);
               }
               for(let sym=1;sym < content[i].insert.toString().length;sym++) {
                 data.blocks.push({
                   depth: "0",
                   entityRanges: [],
                   inlineStyleRanges: [],
                   text: "",
                   type: "unstyled"
                 });
                 len++;
               }
            }
            else {
              var arr = content[i].insert.toString().split('\n');
//              console.log(arr);

//            else {
              for (let j = 0; j < arr.length; j++) {
                //解决文字位置在“换行符”元素处才显示的情况
                if((arr.indexOf("")===j && j===0 && content[i].attributes && len>0 && data.blocks[len-1].text)) {
                  let style = Object.keys(content[i].attributes)[0];
                  style += content[i].attributes[style];
                  data.blocks[len - 1].type = (data.blocks[len - 1].type ==='atomic'?'atomic':style);
                  //解决居中时文字字体不一致的情况
                  if(len>1 && (data.blocks[len-2].text)) {
                    data.blocks[len-2].type = (data.blocks[len - 2].type ==='atomic'?'atomic':style);
                  }
                  continue;
                }
                data.blocks.push({
                  depth: "0",
                  entityRanges: [],
                  inlineStyleRanges: [],
                  text: "",
                  type: "unstyled"
                });
                data.blocks[len].text = arr[j].toString().trim();
                data.blocks[len].inlineStyleRanges.push({
                  length: arr[j].length,
                  style: content[i].attributes
                });
                len++;
              }
            }
//            }
            /*
          //判断是否仅仅是换行符
          if (!content[i].insert.toString().trim() && content[i].attributes) {
            if (i > 0) {
              //attributes放到上个对象的type中
              let style = Object.keys(content[i].attributes)[0];
              style += content[i].attributes[style];
              data.blocks[i - 1].type = style;
            }
            else {
              //未有内容就开始的换行符都忽略
              continue;
            }
          }
          else if (content[i].attributes) {
            data.blocks[i].text = content[i].insert.toString().trim();
            data.blocks[i].inlineStyleRanges.push({
              length: content[i].insert.length,
              style: content[i].attributes
            });
          }
          else if (content[i].insert.image) {
            let name = content[i].insert.image.substring(28, 40);
            let obj = {};
            obj[name] = {
              materialId: 'undefined',//未确定
              name: name,
              url: 'undefined',//未确定
              mutability: 'MUTABLE',
              type: 'SIDERIMAGE'
            };
            data.entityMap[name] = obj;
            data.blocks[i].type = "atomic";
            let enR = {
              key: name,
              length: 1
            };
            data.blocks[i].entityRanges.push(enR);
          }
          else {
            data.blocks[i].text = content[i].insert.toString().trim();
            data.blocks[i].inlineStyleRanges.push({
              length: content[i].insert.length
            });
          }
          */
          }
        }

        console.log(data);
        console.log(JSON.stringify(data));

      }
    },
    computed: {
      editor() {
        return this.$refs.myQuillEditor.quill
      }
    },
    mounted() {
      console.log('this is current quill instance object', this.editor)
    }

  }
</script>

<style>
.quill-editor{
  height:500px;
}
.el-button{
  margin-top: 50px;
}
</style>
