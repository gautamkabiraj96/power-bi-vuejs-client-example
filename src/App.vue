<template>
  <div id="app">
    <section id="container" style="height: 700px" />
  </div>
</template>
<!-- 
www.nightprogrammer.com
-->

<script>
import * as pbi from "powerbi-client";

export default {
  name: "App",
  data() {
    return {
      embedUrl: null,
      accessToken: null,
      sampleReportUrl:
        "https://playgroundbe-bck-1.azurewebsites.net/Reports/SampleReport",
    };
  },
  methods: {
    async initializePowerBI() {
      const sampleReportUrl = this.sampleReportUrl;

      const reportConfigResponse = await fetch(sampleReportUrl);
      if (!reportConfigResponse.ok) {
        console.error("Failed to fetch config for report.");
        console.error(
          `Status: ${reportConfigResponse.status} ${reportConfigResponse.statusText}`
        );
        return;
      }

      const reportConfig = await reportConfigResponse.json();
      console.log(reportConfig);

      console.log("The access token is set. Loading the Power BI report");

      this.embedUrl = reportConfig.EmbedUrl;
      this.accessToken = reportConfig.EmbedToken.Token;
    },
  },
  mounted() {
    this.initializePowerBI().then(() => {
      const permissions = pbi.models.Permissions.All;

      const config = {
        type: "report",
        tokenType: pbi.models.TokenType.Embed,
        accessToken: this.accessToken,
        embedUrl: this.embedUrl,
        pageView: "fitToWidth",
        permissions: permissions,
      };

      let powerbi = new pbi.service.Service(
        pbi.factories.hpmFactory,
        pbi.factories.wpmpFactory,
        pbi.factories.routerFactory
      );

      const dashboardContainer = document.getElementById("container");
      const dashboard = powerbi.embed(dashboardContainer, config);

      dashboard.off("loaded");
      dashboard.off("rendered");
      dashboard.on("error", function () {
        this.dashboard.off("error");
      });
    });
  },
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #000000;
  margin-top: 60px;
}
</style>
