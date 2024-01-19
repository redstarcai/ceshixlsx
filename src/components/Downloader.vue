<template>
  <div @click="download" style="height: 100%; display: flex; justify-content: center; align-items: center;">
    {{ show_text || "导出PDF" }}
  </div>
</template>

<script>
import jsPDF from 'jspdf';
import zionMdapi from "zion-mdapi";

export default {
  name: "PDFExporter",
  props: ["show_text", "task_pk", "actionflow_name", "task_config", "download_config", "filename", "url", "actionflow_id"],
  data() {
    return {
      mdapi: null,
    }
  },
  mounted() {
    this.initMadpi();
  },

  methods: {
    async download() {
      let data;
      if (this.actionflow_name) {
        data = await this.queryDownloadTaskInfo();
      } else {
        data = {
          objects: this?.download_config?.objects,
          schema: this?.download_config?.schema,
          filename: this.filename || 'test.pdf' // Change filename to PDF extension
        }
      }

      if (!data?.schema) {
        data.schema = [{
          content: '测试内容xxx' // Change to the content you want in the PDF
        }]
      }

      const pdf = new jsPDF();
      data.schema.forEach(item => {
        pdf.text(item.content, 10, 10); // Adjust the coordinates as needed
      });

      pdf.save(data?.filename);
    },

    // 获取下载任务信息
    async queryDownloadTaskInfo() {
      const { data, msg, status } = await this.mdapi.callActionflow({
        actionflow_name: this.actionflow_name,
        payload: {
          task_config: this.task_config,
          task_pk: this.task_pk
        }
      }).catch(e => { return { data: {}, msg: e?.message || e, status: "失败" } });

      if (status !== "成功") {
        console.error(msg, data);
      }

      const { objects = [], filename } = data;

      return {
        schema: objects,
        filename
      };
    },

    // 初始化mdapi
    initMadpi() {
      this.mdapi = zionMdapi.init({
        url: this.url,
        actionflow_id: this.actionflow_id,
        env: "H5"
      });
    }
  }
}
</script>

<style scoped>
/* 样式可以根据需要进行定制 */
</style>
