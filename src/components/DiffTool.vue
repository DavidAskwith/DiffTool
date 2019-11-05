<template>
<div class="container">

  <div id="LeftLineNumber" class="lineNumber">
    {{ leftLineNumbers }}</div>
  <div ref="LeftDiff" id="LeftDiff"
    @input="onInput" @paste="onPaste" 
    class="diffInput" contentEditable="true"
    v-html="leftDiffText">
  </div>

  <div id="RightLineNumber" class="lineNumber">
    {{ rightLineNumbers }}
  </div>
  <div ref="RightDiff"
    @input="onInput"  @paste="onPaste" 
    class="diffInput" contentEditable="true"
    v-html="rightDiffText">
  </div>
  <button @click="identifyDiffs">Test</button>
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
    onInput(e) {
        let html = e.target.innerHTML;
        if (e.srcElement.id === "LeftDiff") {
            this.onLeftInput(html);
        } else {
            this.onRightInput(html);
        }
  //      this.identifyDiffs();
    },
    onLeftInput(html) {
        this.leftLineNumbers = '';
        this.leftLineNumbers = this.getLineNumbers(html);
    },
    onRightInput(html) {
        this.rightLineNumbers = '';
        this.rightLineNumbers = this.getLineNumbers(html);
    },

    getLineNumbers(html) {
        let lineNums = '';
        //TODO: follwed by opening or closed div to identify frist line
        let totalLineNums = (html.match(/((<div>).*?(<\/div>))|(^.*?(?=(?:<div>)))/mg) || [1]).length;

        for (let i = 1; i <= totalLineNums; i++) {
            lineNums += `${i}\n`;
        }

        return lineNums;
    },
    identifyDiffs(){
        let leftDiffText = '';
        let rightDiffText = '';
        //TODO: not br and not followed by br
        var regex = /(?:(?:<div>)(.*?(?!<br>))(?:<\/div>))|(?:(^.*?)(?=<div>))/mg;
        let leftLines = this.$refs.LeftDiff.innerHTML.replace(/(<span>)|(<\/span>)/mg, '').matchAll(regex);
        let rightLines = this.$refs.RightDiff.innerHTML.replace(/(<span>)|(<\/span>)/mg, '').matchAll(regex)

        console.log(rightLines)
        // will need to iterate through for whichever side is longer
        for(const leftLine of leftLines) {
            let rightLine = null;
            let rightContent;
            let leftContent = leftLine[1] || leftLine[2]

            let nextLine = rightLines.next()
            if(!nextLine.done) {
                console.log(rightLines)
                rightLine = nextLine.value;
                rightContent = rightLine[1] || rightLine[2];
            }

            console.log(leftContent)
            console.log(rightContent)
            if(rightContent === leftContent) {
                leftDiffText += `<div>${leftContent || ''}</div>`;
                rightDiffText += `<div>${rightContent || ''}</div>`;
            } else {
                let diffedLines = this.getDiffsInLine((leftContent || ''), (rightContent || ''));
                leftDiffText += `<div>${diffedLines.leftLine || ''}</div>`;
                rightDiffText += `<div>${diffedLines.rightLine || ''}</div>`;
            }


        }
        this.leftDiffText = leftDiffText;
        this.rightDiffText = rightDiffText;
    },
    getDiffsInLine(leftContent, rightContent) {
        //TODO: iterate based on longest!!
        let rightLetters = rightContent.split('');
        console.log(":dsflk")

        let leftLetters = leftContent.split('').map((letter, index) => {
            console.log(letter)
            if (letter == (rightLetters[index] || '')) {
                return letter;
            } else {
                let curRightLetter = rightLetters[index]
                rightLetters[index] = curRightLetter == undefined ? null :  `<span>${curRightLetter}</span>`;
                return `<span>${letter}</span>`;
            }
        });
        return { leftLine : leftLetters.join(''), rightLine : rightLetters.join('') };

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
/deep/ span {
    background-color: red;
}
</style>
