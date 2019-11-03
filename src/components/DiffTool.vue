<template>
<div class="container">

  <div id="LeftLineNumber" class="lineNumber">
    {{ leftLineNumbers }}</div>
  <div id="LeftDiff" @input="onLeftInput" @paste="onPaste" class="diffInput" contentEditable="true">
    {{ leftDiffText }}
  </div>

  <div id="RightLineNumber" class="lineNumber">
    {{ rightLineNumbers }}
  </div>
  <div id="RightDiff" @input="onRightInput"  @paste="onPaste" class="diffInput" contentEditable="true">
    {{ rightDiffText }}
  </div>

</div>
</template>

<script>
export default {
  name: 'DiffTool',

  props: {
  },

  data: function () {
    return {
      leftDiffText: '',
      rightDiffText: '',
      leftLineNumbers: '1',
      rightLineNumbers: '1',
    }
  },

  methods: {
    onPaste(e) {
        e.preventDefault();

        var text = (e.originalEvent || e).clipboardData.getData('text/plain');
        document.execCommand("insertText", true, text);
    },
    onLeftInput(e) {

        let html = e.target.innerHTML;

        this.leftLineNumbers = '';
        this.leftLineNumbers = this.getLineNumbers(html);
    },
    onRightInput(e) {
        this.rightLineNumbers = this.getLineNumbers(html);
        let html = e.target.innerHTML;
        this.rightLineNumbers = '';
        this.rightLineNumbers = this.getLineNumbers(html);
    },

    getLineNumbers(html) {
        let lineNums = '';
        let totalLineNums = (html.match(/((<div>).*?(<\/div>))|(^\S*?(?=<div>))/mg) || [1]).length;
        console.log(html.match(/(<div>.*?<\/div>)|(^\S*?(?=<div>))/g));

        for (let i = 1; i <= totalLineNums; i++) {
            lineNums += `${i}\n`;
            console.log(i);
        }

        return lineNums;
    },
  }
}
</script>

<style scoped>
.container {
  display: flex;
  min-height: 400px;
  margin: 0;
}

.diffInput {
  border: 1px solid grey;
  width: 50%;
  margin: 15px 15px 15px 0;
  background-color: white;
  white-space: nowrap;
  overflow-x: scroll;
}

.lineNumber {
  width: 20px;
  background: darkgrey;
  margin: 15px 0;
  border-style: solid;
  border-color: grey;
  border-width: 1px 0 1px 1px;
  text-align: center;
}

#LeftLineNumber {
  margin-left: 15px;
}
</style>
