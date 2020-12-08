<template>
  <div>
    <input
      @input="handleChange"
      @focus="handleFocus"
      @blur="handleBlur"
      @keydown="buttunCorrect"
      @click="clickInput"
      @paste="onPaste"
      @select="onSelect"
      type="text"
      name="phone"
      value="+7 (XXX) XXX-XX-XX"
      autocomplete="off"
      class="input-phone input-normal"
      :class="{
        'input-focus-with-text': phone && isFocused,
        'input-focus-without-text': !phone && isFocused,
        'input-not-correct':
          isNotCorrect ||
          (blurNotFullPhone && phone.replace(/\D+/g, '').length < 11),
        'with-text': phone.length,
        'style-without-text-developer': !phone && otherStyleWithoutText === 'styleWithoutTextDeveloper',
        'style-without-text-residentional': !phone && otherStyleWithoutText === 'styleWithoutTextResidentional',
        'style-with-text-developer': phone.length && otherStyleWithText === 'styleWithTextDeveloper',
        'style-with-text-residentional': phone.length && otherStyleWithText === 'styleWithTextResidentional',

      }"
    />
  </div>
</template>
<script>
export default {
  data() {
    return {
      matrix: "(XXX) XXX-XX-XX",
      matrixForCopyPast: "(XXX) XXX-XX-XX",
      phone: "",
      keyNumber: "",
      paste: false,
      buttonBackspacePress: false,
      buttonDeletePress: false,
      selectionStart: "",
      selectionEnd: "",
      cursorPosition: "4",
      isFocused: false,
      blurNotFullPhone: false,
    };
  },
  props: {
    otherStyleWithText:{
            type: String,
            required: true,
            default() {
              return ''
            }
    },
    otherStyleWithoutText:{
            type: String,
            required: true,
            default() {
              return ''
            }
    },

    isNotCorrect: {
      type: Boolean,
      default: false,
    },
  },
  methods: {
    onSelect(event) {
      let et = event.target;
      this.selectionStart = et.selectionStart;
      this.selectionEnd = et.selectionEnd;
    },
    onPaste() {
      this.paste = true;
    },
    clickInput() {
      let et = event.target;
      if (et.value == et.defaultValue) {
        this.setCursorPosition("4", et);
      }
    },

    buttunCorrect(event) {
      let cursorPosition = (this.cursorPosition = this.getCursorPosition(
        event.target
      ));
      let setCursorPosition;
      let et = event.target;
      this.keyNumber = event.key;
      if (event.key === "Delete") {
        if (this.selectionStart == "0" && this.selectionEnd == "18") {
          this.matrix = this.matrixForCopyPast;
        }
        this.buttonDeletePress = true;
      } else {
        this.buttonDeletePress = false;
      }
      if (event.key === "Backspace") {
        if (cursorPosition == 9) {
          setCursorPosition = 7;
        } else if (cursorPosition == 8) {
          setCursorPosition = 7;
        } else if (cursorPosition == 13 || cursorPosition == 16) {
          setCursorPosition = cursorPosition - 1;
        } else {
          setCursorPosition = cursorPosition;
        }
        this.setCursorPosition(setCursorPosition, et);

        if (this.selectionStart == "0" && this.selectionEnd == "18") {
          event.target.value = "+";
        }
        this.buttonBackspacePress = true;
      } else {
        this.buttonBackspacePress = false;
      }
    },
    setCursorPosition(pos, elem) {
      if (elem.setSelectionRange) {
        elem.setSelectionRange(pos, pos);
      }
    },
    getCursorPosition(eventTarg) {
      let caretPos;
      caretPos = eventTarg.selectionStart;
      return caretPos;
    },

    handleChange(event) {
      let evtarget = event.target;
      let defaultValue = evtarget.value;
      let keyButton = this.keyNumber;
      let newVal = "";
      let i = 4;
      let cursorPosition = this.getCursorPosition(event.target);
      let setCursorPosition = "4";
      let clearCopyNumber = defaultValue
      let serchPlus = clearCopyNumber.match(/\+/g) || [];
      keyButton.toString();
      if (!isNaN(keyButton)) {
        //отфильтруем введенные в инпут символы, оставим числа
        newVal = keyButton;
      }

      clearCopyNumber = clearCopyNumber.replace(/\D/g, "");//получаем чистые цыфры из вставленного в инпут текста
      clearCopyNumber = clearCopyNumber.split("");
      clearCopyNumber.splice(0, 1); //убираем из номера цифру 7 попавшую туда из значения value
      if(serchPlus.length > 1){
        clearCopyNumber.splice(0, 1); //убираем из номера цифру 7 если юзер скопировал телефон вместе с +7
      }else{
          if (clearCopyNumber[0] == "8") {//проверяем есть ли в скопированном номере первая цифра 8
            clearCopyNumber.splice(0, 1); //удаляем ее
          }
      }

      if (this.paste) {//если произошло событие paste (копирование)

        let matrixNumber = this.matrixForCopyPast.split("");
        for (let i = 0; i < matrixNumber.length; i++) {
          let symbolMatrix = matrixNumber[i];
          if (symbolMatrix == "X") {
            if (clearCopyNumber.length != 0) {
              matrixNumber[i] = clearCopyNumber[0];
              clearCopyNumber.splice(0, 1);
            } else {
              break;
            }
          }
        }
        let newArreyNumber = matrixNumber.join("");
        this.matrix = newArreyNumber;
        this.paste = false;
      }
      if (                                     //ввод цыфр буквы удаляются
        this.buttonBackspacePress === false &&
        this.buttonDeletePress === false
      ) {

        if (cursorPosition == 5 && newVal == "8") {
          newVal = "X";
        }
        this.matrix = this.matrix.replace(/./g, function (symbol) {
          let mainReturnSymbol;
          if (symbol == "X" && i == cursorPosition && newVal != "") {
            mainReturnSymbol = newVal;
          } else if (symbol == "X" && i != cursorPosition) {
            mainReturnSymbol = symbol;
          } else if (symbol != isNaN) {
            mainReturnSymbol = symbol;
          }
          i++;
          return mainReturnSymbol;
        });
        i = 0;
        if (cursorPosition <= 4) {
          setCursorPosition = "4"; //установим позицию курсора после первой скобки
          //если юзер пытается ввести что то перед скобкой или перед +7
          this.matrix = this.matrix.replace(/./g, function (symbol) {
            let mainReturnSymbol;
            if (symbol == "X" && i == 1 && newVal != "") {
              mainReturnSymbol = newVal;
            } else if (symbol == "X" && i != cursorPosition) {
              mainReturnSymbol = symbol;
            } else if (symbol != isNaN) {
              mainReturnSymbol = symbol;
            }
            i++;
            return mainReturnSymbol;
          });
        } else {
          setCursorPosition = cursorPosition; //перемещаем курсор за введенный символ
        }

        if (cursorPosition == 7) {
          setCursorPosition = cursorPosition + 2; //перекидываем курсор через скобку и пробел
        } else if (
          cursorPosition == 9 ||
          cursorPosition == 12 ||
          cursorPosition == 15
        ) {
          setCursorPosition = cursorPosition + 1; //перекидываем курсор через тире
        }
        if (newVal == "X") {
          setCursorPosition = cursorPosition - 1;
        }
        if (newVal == "") {
          setCursorPosition = cursorPosition - 1;
        }
      } else if (this.buttonBackspacePress === true) { // если нажата  кнопка Backspase

        if (this.selectionStart == this.selectionEnd) {
          this.matrix = this.matrix.replace(/./g, function (symbol) {
            let mainReturnSymbol;
            if (symbol == "(") {
              mainReturnSymbol = symbol;
            } else if (symbol == "X") {
              mainReturnSymbol = symbol;
            } else if (symbol != isNaN && i == cursorPosition + 1) {
              mainReturnSymbol = "X";
            } else if (symbol != isNaN) {
              mainReturnSymbol = symbol;
            }
            i++;
            return mainReturnSymbol;
          });
          if (cursorPosition <= 4) {
            setCursorPosition = "4"; //установим позицию курсора после первой скобки
          } else {
            //если юзер пытается удалить первую скобку и +7

            setCursorPosition = cursorPosition; //перемещаем курсор за введенный символ
          }
          if (cursorPosition == 9) {
            setCursorPosition = cursorPosition - 2; //перекидываем курсор через скобку и пробел
          } else if (
            cursorPosition == 8 ||
            cursorPosition == 13 ||
            cursorPosition == 16
          ) {
            setCursorPosition = cursorPosition - 1; //перекидываем курсор через тире
          }
        } else {
          //выделен весь телефон
          if (this.selectionStart == "0") {
            this.matrix = this.matrixForCopyPast;
          } else {
            //если выделена часть телефона

            this.matrix = this.matrix.replace(
              /./g,
              function (symbol) {
                let mainReturnSymbol;

                if (symbol == "(") {
                  mainReturnSymbol = symbol;
                } else if (symbol == "X") {
                  mainReturnSymbol = symbol;
                } else if (symbol == ")") {
                  mainReturnSymbol = symbol;
                } else if (symbol == " ") {
                  mainReturnSymbol = symbol;
                } else if (symbol == "-") {
                  mainReturnSymbol = symbol;
                } else if (
                  symbol != isNaN &&
                  i - 1 >= this.selectionStart &&
                  i - 1 < this.selectionEnd
                ) {
                  mainReturnSymbol = "X";
                } else if (symbol != isNaN && i - 1 < this.selectionStart) {
                  mainReturnSymbol = symbol;
                } else if (symbol != isNaN && i - 1 >= this.selectionStart) {
                  mainReturnSymbol = symbol;
                }
                i++;
                return mainReturnSymbol;
              }.bind(this)
            );
          }

          if (cursorPosition <= 4) {
            setCursorPosition = "4"; //установим позицию курсора после первой скобки
          } else {
            //если юзер пытается удалить первую скобку и +7

            setCursorPosition = cursorPosition; //перемещаем курсор за введенный символ
          }
          if (this.selectionStart == 9) {
            setCursorPosition = cursorPosition - 1; //перекидываем курсор через скобку и пробел
          } else if (
            cursorPosition == 8 ||
            cursorPosition == 13 ||
            cursorPosition == 16
          ) {
            setCursorPosition = cursorPosition - 1; //перекидываем курсор через тире
          }
        }
        this.buttonBackspacePress === false;
      } else if (this.buttonDeletePress === true) {  //если нажата кнопка Delite


        if (this.selectionStart == this.selectionEnd) {
          //удаляем по одному символу
          this.matrix = this.matrix.replace(
            /./g,
            function (symbol) {
              let mainReturnSymbol;
              if (symbol == "(") {
                mainReturnSymbol = symbol;
              } else if (symbol == ")") {
                mainReturnSymbol = symbol;
              } else if (symbol == " ") {
                mainReturnSymbol = symbol;
              } else if (symbol == "-") {
                mainReturnSymbol = symbol;
              } else if (symbol == "X") {
                mainReturnSymbol = symbol;
              } else if (symbol != isNaN && i == cursorPosition + 1) {
                mainReturnSymbol = "X";
              } else if (symbol != isNaN) {
                mainReturnSymbol = symbol;
              }
              i++;
              return mainReturnSymbol;
            }.bind(this)
          );
          if (cursorPosition < 4) {
            setCursorPosition = "4"; //установим позицию курсора после первой скобки
          } else {
            //если юзер пытается удалить первую скобку и +7

            setCursorPosition = cursorPosition + 1; //перемещаем курсор за введенный символ
          }
          if (cursorPosition == 6) {
            setCursorPosition = cursorPosition + 3; //перекидываем курсор через скобку и пробел
          } else if (cursorPosition == 11 || cursorPosition == 14) {
            setCursorPosition = cursorPosition + 2; //перекидываем курсор через тире
          }
        } else {
          // удаляем выделенные части телефона
          this.matrix = this.matrix.replace(
            /./g,
            function (symbol) {
              let mainReturnSymbol;
              if (symbol == "(") {
                mainReturnSymbol = symbol;
              } else if (symbol == "X") {
                mainReturnSymbol = symbol;
              } else if (symbol == ")") {
                mainReturnSymbol = symbol;
              } else if (symbol == " ") {
                mainReturnSymbol = symbol;
              } else if (symbol == "-") {
                mainReturnSymbol = symbol;
              } else if (
                symbol != isNaN &&
                i - 1 >= this.selectionStart &&
                i - 1 < this.selectionEnd
              ) {
                mainReturnSymbol = "X";
              } else if (symbol != isNaN && i - 1 < this.selectionStart) {
                mainReturnSymbol = symbol;
              } else if (symbol != isNaN && i - 1 >= this.selectionStart) {
                mainReturnSymbol = symbol;
              }
              i++;
              return mainReturnSymbol;
            }.bind(this)
          );
        }
        this.buttonDeletePress === false;
      }
      event.target.value = "+7 " + this.matrix; //отрисовываем в инпут

      this.setCursorPosition(setCursorPosition, evtarget);
      this.blurNotFullPhone = false;

      this.phone = event.target.value;
      if(event.target.value === "+7 (XXX) XXX-XX-XX"){
          this.phone = ""
      }
      this.$emit("input", this.phone.replace(/\D+/g, "")); // убираем все символы кроме цифр.

      //-----------ЕСЛИ ОСТАЛИСЬ ЦИФРЫ ОНИ ДОЛЖНЫ ПЕРЕПИСЫВАТЬСЯ РЕАЛИЗОВАТЬ
    },

    handleFocus() {
      this.isFocused = true;
    },
    handleBlur() {
      this.isFocused = false;
      this.blurNotFullPhone = true;
    },
  },

  updated() {},
  mounted() {},
};
</script>




<style lang="scss" scoped>
.input-normal {
  display: flex;
  font-size: 14px;
  line-height: 21px;
  color:rgba(255, 255, 255, 0.6);
  background:white;
  border: 1px solid #D7DDEB;
  border-radius: 4px;
  width: 100%;
  height: 40px;
  margin-right: 10px;
  padding-left: 21%;
  padding-right: 10px;

}

.input-focus-with-text {
  border: 1.5px solid rgba(1, 121, 209, 0.65);
}
.input-focus-without-text {
  border: 1.5px solid rgba(230, 70, 70, 0.6);
}
.input-not-correct {
  border: 1.5px solid rgba(230, 70, 70, 0.6) !important;
}
.with-text {
  color: white;
}
.style-without-text-developer{
    color: rgba(125, 131, 141, 0.6);
}
.style-without-text-developer:hover{
    color: rgba(125, 131, 141, 0.6);
}
.style-with-text-developer{
    color:#3D3D3D;
}
.style-with-text-developer:hover{
    color:#3D3D3D;
}
.style-without-text-residentional{
    color: #a6d0ee;
    background: rgba(255, 255, 255, 0.15);
    border: 1px solid rgba(255, 255, 255, 0.3);
}
.style-without-text-residentional:hover{
    color: white;

}
.style-with-text-residentional{
    color:white;
    border: 1px solid rgba(255, 255, 255, 0.3);
    background: rgba(255, 255, 255, 0.15);
}
.style-with-text-residentional:hover{
    color:white;
}
</style>
