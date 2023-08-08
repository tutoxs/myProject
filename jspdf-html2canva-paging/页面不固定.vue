<template>
  <div class="ifPagingNode">222</div>
</template>

<script>
export default {
  props: {
    height: {
      type: [Number, String],
      default: "760",
    },
    reportData: {
      type: Object,
      default: () => {
        return {};
      },
    },
  },
  data() {
    return {
      pageSum: 0,
      contentPaiing: "0",
    };
  },
  methods: {
    computPage() {
      this.$nextTick(() => {
        if (this.$refs.commentBox) {
          // 根据自己的页面算出页数
          let commentBoxHeight = this.$refs.commentBox.offsetHeight;
          let commentContent1BoxHeight =
            this.$refs.commentContent1Box.offsetHeight;
          let baseHeight = Math.ceil((commentContent1BoxHeight - 24) / 24) * 24;
          let otherHeight = commentBoxHeight - baseHeight;
          let pageHeight = Math.floor((this.height * 0.886 - 24) / 24) * 24;
          let page = Math.ceil(otherHeight / pageHeight);
          this.pageSum = page;
          // 表格的下边位置
          this.getAllNodes();
        }
      });
    },
    getAllNodes() {
      // 需判断分页的节点
      const lableListID = document.querySelectorAll(".ifPagingNode");
      // 获取到第一页的位置信息
      const pageHeight = this.$refs.contenxtPage.getBoundingClientRect();
      // 处理每一页分割位置
      let pageInfo = [];
      for (let i = 0; i < this.pageSum + 1; i++) {
        pageInfo.push(Math.ceil(pageHeight.bottom + this.height * i));
      }
      pageInfo.forEach((item) => {
        for (let i = 0; i < lableListID.length; i++) {
          // 每个元素的bottom位置
          const positionInfo = lableListID[i].getBoundingClientRect();
          const elementBottom = Math.ceil(positionInfo.bottom);
          if (elementBottom > item - 24) {
            // 新div的高度
            const unllHei =
              item -
              24 -
              18 -
              lableListID[i - 1].getBoundingClientRect().bottom;
            this.createEmptyElement(
              lableListID[i],
              unllHei,
              pageHeight,
              lableListID[lableListID.length - 1]
            );
            break;
          }
        }
      });
    },
    createEmptyElement(currentNode, unllHei, pageHeight, endeNode) {
      if (Math.sign(unllHei) == -1) {
        unllHei = 24;
      }
      // 获取该div的父节点,创建新节点，设置样式
      let divParent = currentNode.parentNode;
      let newNode = document.createElement("div");
      newNode.className = "emptyDiv";
      newNode.style.height = unllHei + "px";
      newNode.style.width = "100%";
      newNode.style.marginBottom = 24 + 24 + 18 + "px";
      newNode.style.overflow = "hidden";
      newNode.style.paddingLeft = "12px";
      newNode.style.paddingRight = "12px";
      newNode.style.boxSizing = "borderBox";

      // 是否为文字标签
      if (
        currentNode.childNodes[0].nodeName.toLowerCase() === "#text" &&
        currentNode.offsetHeight !== 40
      ) {
        const context = currentNode.innerText;
        newNode.innerText = context;
        // 文字信息
        const textInfo = window.getComputedStyle(currentNode, null);
        const lineHe = textInfo.lineHeight.split("p");

        // 重置容器的高
        const container = lineHe[0] * Math.floor(unllHei / lineHe[0]);
        newNode.style.height = container + "px";
        newNode.style.marginBottom =
          24 + 24 + 18 + (unllHei - container) + "px";
        // 获取剩余的盒子高
        const currHeight = textInfo.height.split("p")[0] - container + "px";
        currentNode.style.height = currHeight;

        if (container > 24) {
          newNode.innerText = currentNode.innerText;
          let newTextNode = document.createElement("div");
          newTextNode.innerText = currentNode.innerText;
          currentNode.innerText = "";
          newTextNode.style.position = "relative";
          newTextNode.style.top = -container + "px";
          currentNode.appendChild(newTextNode);
        } else {
          console.log(container, currentNode.offsetHeight, "12345");
          currentNode.style.height = "24px";
          newNode.style.marginBottom = 24 + (unllHei - container) + "px";
        }
        currentNode.style.overflow = "hidden";
      }
      // 将新节点插入到div的下一个兄弟节点之前，即div之后
      divParent.insertBefore(newNode, currentNode);

      // 内容总高
      const content = Math.ceil(
        endeNode.getBoundingClientRect().bottom - pageHeight.bottom
      );
      this.pageSum = Math.ceil(content / this.height);
      /* 执行下载并打印PDF的方法 */
      this.pdfs(pdfDom, title);
    },
    pdfs() {
      //div内部滚动导致内容不全处理
      document.getElementById("app").style.height = "auto";
      setTimeout(() => {
        html2canvas(document.getElementById("reportBox"), {
          background: "#01195e",
          allowTaint: true,
          useCORS: true,
          // height: document.getElementById('upload').scrollHeight,
          // windowHeight: document.getElementById('upload').scrollHeight
        }).then((canvas) => {
          // console.log(canvas);
          // var contentWidth = canvas.width;
          // var contentHeight = canvas.height;
          var contentWidth = parseInt(canvas.style.width) * 2;
          var contentHeight = parseInt(canvas.style.height) * 2;

          // console.log('contentWidth', contentWidth);
          // console.log('contentHeight', contentHeight);
          //一页pdf显示html页面生成的canvas高度;
          var pageHeight = (contentWidth / 592.28) * 841.89;
          //未生成pdf的html页面高度
          var leftHeight = contentHeight;
          //页面偏移
          var position = 0;
          //a4纸的尺寸[595.28,841.89]，html页面生成的canvas在pdf中图片的宽高
          var imgWidth = 595.28;
          var imgHeight = (592.28 / contentWidth) * contentHeight;
          // console.log('imgHeight', imgHeight);
          var pageData = canvas.toDataURL("image/jpeg", 1.0);

          var pdf = new jsPDF("", "pt", "a4");

          //有两个高度需要区分，一个是html页面的实际高度，和生成pdf的页面高度(841.89)
          //当内容未超过pdf一页显示的范围，无需分页
          if (leftHeight < pageHeight) {
            pdf.addImage(pageData, "JPEG", 0, 0, imgWidth, imgHeight);
          } else {
            while (leftHeight > 0) {
              pdf.addImage(pageData, "JPEG", 0, position, imgWidth, imgHeight);
              leftHeight -= pageHeight;
              position -= 841.89;
              //避免添加空白页
              if (leftHeight > 100) {
                pdf.addPage();
              }
            }
          }
        });
      }, 300);
    },
  },
};
</script>
