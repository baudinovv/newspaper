<template>
  <div>
    <canvas 
      class="w-full"
      ref="imageCanvas">
    </canvas>
    <br>
    <div class="flex flex-wrap">
      <textarea 
        v-model="newsName" class="text-center max-w-[290px] font-bold italic text-5xl"  
        placeholder="Введите Название газеты" rows="5" cols="40"></textarea>
      <textarea 
        v-model="userTitle" class="text-justify max-w-[290px] font-bold italic text-5xl"  
        placeholder="Введите Заголовок 1" rows="5" cols="40"></textarea>
      <textarea 
        v-model="userTitle2" class="text-justify max-w-[290px] font-bold italic text-5xl"  
        placeholder="Введите Заголовок 2" rows="5" cols="40"></textarea>
      <textarea 
        v-model="userText" class="text-justify max-w-[290px]"  placeholder="Введите текст 1" rows="5" cols="40"></textarea>
      <textarea 
        v-model="userText2" class="text-justify max-w-[290px]"  placeholder="Введите текст 2" rows="5" cols="40"></textarea>
      <textarea 
        v-model="userText3" class="text-justify max-w-[290px]"  placeholder="Введите текст 3" rows="5" cols="40"></textarea>
    </div>
    <div class="">
      <button class="mr-5 transition px-2 py-2 border rounded hover:bg-black hover:text-white" @click="drawAll">Добавить текст</button>
      <button class="mr-5 transition px-2 py-2 border rounded hover:bg-black hover:text-white" @click="downloadImage">Скачать картинку</button>
      <button class="mr-5 transition px-2 py-2 border rounded hover:bg-black hover:text-white" @click="clearText">Удалить весь текст</button>
    </div>
  </div>
</template>

<script lang="ts">
export default {
  data() {
    return {
      newsName: '',
      userTitle: '',
      userTitle2: '',
      userText: '',
      userText2: '',
      userText3: '',
      image: null,
    };
  },
  methods: {
    clearText() {
      const canvas = this.$refs.imageCanvas;
      const context = canvas.getContext('2d');
      context.clearRect(0, 0, canvas.width, canvas.height);

      if (this.image) {
        context.drawImage(this.image, 0, 0);
      }
    },
    drawText(text, x, y, lineHeight, maxWidth, fontStyle, justifyMode) {
      const canvas = this.$refs.imageCanvas;
      const context = canvas.getContext('2d');

      // Устанавливаем стили текста
      console.log(fontStyle)
      context.font = fontStyle;
      context.fillStyle = 'black';

      // Разделяем текст на строки
      const lines = this.wrapText(context, text, maxWidth, lineHeight);

      // Рисуем строки с проверкой на последнюю строку абзаца
      lines.forEach((line, index) => {
        const isLastLine = index === lines.length - 1 || lines[index + 1] === ''; // Проверка последней строки абзаца
        if (isLastLine && justifyMode) {
          // Последняя строка — обычное выравнивание
          context.fillText(line, x, y);
        } else {
          // Остальные строки — выравнивание по ширине
          this.justifyText(context, line, x, y, maxWidth); 
        }
        y += lineHeight;
      });
    },
    wrapText(context, text, maxWidth, lineHeight) {
      const lines = text.split('\n');
      const wrappedLines = [];

      lines.forEach(line => {
        if (line === '') {
          wrappedLines.push('');
        } else {
          const words = line.split(' ');
          let currentLine = '';

          words.forEach(word => {
            const testLine = currentLine + word + ' ';
            const testWidth = context.measureText(testLine).width;

            if (testWidth > maxWidth && currentLine !== '') {
              wrappedLines.push(currentLine);
              currentLine = word + ' ';
            } else {
              currentLine = testLine;
            }
          });

          if (currentLine !== '') {
            wrappedLines.push(currentLine);
          }
        }
      });

      return wrappedLines;
    },
    justifyText(context, line, x, y, maxWidth) {
      const words = line.trim().split(' ');
      const lineWidth = context.measureText(line.trim()).width;

      if (words.length === 1) {
        context.fillText(line, x, y);
        return;
      }

      const spaceWidth = (maxWidth - lineWidth) / (words.length - 1);
      let currentX = x;

      words.forEach((word, index) => {
        context.fillText(word, currentX, y);

        if (index < words.length - 1) {
          currentX += context.measureText(word).width + spaceWidth;
        }
      });
    },
    downloadImage() {
      if (!this.image) {
        alert('Сначала загрузите изображение');
        return;
      }
      const canvas = this.$refs.imageCanvas;
      const link = document.createElement('a');
      link.download = 'image-with-text.png';
      link.href = canvas.toDataURL();
      link.click();
    },
    drawAll() {
      this.drawText(this.userText, 50, 650, 20, 290, "14px IBM Plex Serif", true);
      this.drawText(this.userText2, 380, 400, 20, 290, "14px IBM Plex Serif", true);
      this.drawText(this.userText3, 690, 400, 20, 290, "14px IBM Plex Serif", true);
      this.drawText(this.userTitle, 380, 315, 40, 290, "italic bold 48px IBM Plex Serif", false);
      this.drawText(this.userTitle2, 690, 315, 40, 290, "italic bold 48px IBM Plex Serif", false);
      this.drawText(this.newsName, 400, 130, 20, 290,"60px Chomsky", false);
    },
    loadImg() {
      const img = new Image();
      img.src = "src/assets/turkey.jpg";
      img.onload = () => {
        const canvas = this.$refs.imageCanvas;
        const context = canvas.getContext('2d');
        canvas.width = img.width;
        canvas.height = img.height;
        context.drawImage(img, 0, 0);
        this.image = img;
      };
    }
  },
  mounted() {
    this.loadImg();
    this.drawAll();
  }
};


</script>

<style scoped>
*{
  font-family: "IBM Plex Serif";
}
@font-face {
  font-family: 'Chomsky';
  font-style: normal;
  font-weight: 400;
  /* Браузер сначала попробует найти шрифт локально */
  src: url("./assets/fonts/Chomsky-399c.woff") format("woff");
}
canvas {
  border: 1px solid black;
}
</style>
