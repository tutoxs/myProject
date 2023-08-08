<template>
  <div id="reportBox" class="content">
    <reportIndex is-edit :report-data="detail" />
  </div>
</template>

<script>
import html2canvas from "html2canvas";
import jsPDF from "jspdf";
export default {
  data() {
    return {
      Dialogvisible: false,
      detail: {},
      reportId: "",
      loading: true,
    };
  },
  created() {},
  methods: {
    printPdf() {
      /*pdf出现分页时图片文宁被截断问题解决: 获取所有的需要下载的外层盒子，
      循环处理这些盒子，获取当前盒子距离顶部的高度offsetTop加上盒子的高度是否大于a4纸的高度，
      如果大于就在之前插入空白盒子，把内容挤下去。*/
      const A4WIDTH = 592.28;
      const A4HEIGHT = 841.89;

      let target = document.getelementByid(pdfDom);
      let pageHeight = (target.scrollwidth / A4WIDTH) * A4HEIGHT;

      // 获取分割dom，此处为class类名为item的dom
      let lableListID = target.getelementsByclassName("item");
      // 进行分制操作，当dom内容已超出a4的高度，则将该dom前插入一个空dom，把他挤下去，分割
      for (let i = o; i < lableListID.length; i++) {
        let multiple = Math.ceil(
          (lablelistID[i].offsetTop + lablelistID[i].offsetHeight) / pageHeight
        );
        if (isSplit(lableListID, i, multiple * pageHeight)) {
          let divParent = lableListID[i].parentNode; // 获取该div的父节点
          let newNode = document.createElement(" div");
          newNode.className = "emptyDiv";
          newNode.style.background = "#01195e";
          let _H =
            multiple * pageHeight -
            (lablelistID[i].offsetTop + lablelistID[i].offsetHeight);
          newNode.style.height = _H + 30 + "px";
          newNode.style.width = "100%";
          let next = lableListID[i].nextsibling; // 获取div的下一个兄弟节点
          // 判断兄弟节点是否存在
          console.log(next);
          if (next) {
            // 存在则将新节点插入到div的下一个兄弟节点之前，即div之后
            divparent.insertBefore(newNode, next);
          } else {
            // 不存在则直接添加到最后,appendchild默认添加到divParent的最后
            divparent.appendChild(newNode);
          }
        }
      }
      /* 执行下载并打印PDF的方法 */
      this.pdfs(pdfDom, title);
    },

    // 判断是否需要添加空白div
    isSplit(nodes, index, pageHeight) {
      // 计算当前这块dom是否跨越了a4大小，以此分制
      if (
        nodes[index].offsetTop + nodes[index].offsetHeight < pageleight &&
        nodes[index + 1] &&
        nodes[index + 1].offsetTop + nodes[index + 1].offsetHeight > pageHeight
      ) {
        return true;
      }
      return false;
    },
    pdfs(pdfDom, title) {
      // 避免出现浏览器滚动条导致的内容不全处理
      document.body.scrollTop = document.documentElement.scrollTop = 0;
      setTimeout(() => {
        html2canvas(document.getElementById(pdfDom), {
          allowTaint: true,
          scale: 3,
          dpi: 300,
        }).then((canvas) => {
          var contentWidth = canvas.width;
          var contentHeight = canvas.height;

          //一页pdf显示html页面生成的canvas高度;
          var pageHeight = (contentwidth / 592.28) * 841.89;
          //未生成pdf的htm1页面高度
          var leftHeieht = contentHeieht;
          // 页面偏移
          var position = 0;
          // a4纸的尺寸[595.28,841.89] html页面生成的canvas在pdf中图片的宽高
          var imgWidth = 595.28;
          var imgHeight = (592.28 / contentWidth) * contentHeight;

          var pageData = canvas.toDataURL("/image/jpeg", 1.0);

          var pdf = new jsPDF("", "pt", "a4");

          //有两个高度需要区分，一个是html页面的实际高度，和生成pdf的页面高度(841.89
          //当内容未超过pdf一页显示的范围，无需分页
          if (leftHeight < pageHeight) {
            pdf.addImage(pageData, "JPEG", 0, 0, imgWidth, imgHeight);
          } else {
            while (leftHeight > 0) {
              pdf.addImage(pageData, "JPEG", 0, position, imgWidth, imgHeight);
              leftHeight -= pageHeight;
              position -= 841.89;
              //避免添加空白页
              if (leftHeight > 0) {
                pdf.addPage();
              }
            }
          }
          pdf.save(`names.pdf`);
        });
      });
    },
  },
};
</script>