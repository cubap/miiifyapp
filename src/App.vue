<template>
  <Header />
  <div class="container">
    <div>
      <img id="desk" src="./assets/desk.jpg" />
    </div>
    <link
      rel="stylesheet"
      href="https://cdn.jsdelivr.net/npm/@recogito/annotorious@2.7.1/dist/annotorious.min.css"
    />
  </div>
  <Footer @button-load="getAnnotations" />
</template>

<script>
import Header from "./components/Header";
import Footer from "./components/Footer";

import { Annotorious } from "@recogito/annotorious";

const axios = require("axios");

export default {
  name: "App",
  components: {
    Header,
    Footer,
  },
  data() {
    return {
      anno: [],
    };
  },
  methods: {
    async createContainer() {
      console.log("createContainer");
      const container = {
        "@context": [
          "http://www.w3.org/ns/anno.jsonld",
          "http://www.w3.org/ns/ldp.jsonld",
        ],
        type: ["BasicContainer", "AnnotationCollection"],
        label: "A demo container",
      };
      const headers = {
        "Slug": "demo",
      };
      try {
        const res = await axios.post("/annotations/", container, { headers: headers });
        console.log(res);
        return res;
      } catch (err) {
        // Handle Error Here
        console.error(err);
      }
    },
    async getAnnotations() {
      console.log("getAnnotations");
      try {
        const res = await axios.get("/annotations/demo/");
        self.anno.setAnnotations(res.data.first.items);
      } catch (err) {
        // if there is no container we need to create one
        if (err.response.status == 404) {
          this.createContainer();
        } else {
          console.error(err);
        }
      }
    },
    async postAnnotation(annotation) {
      console.log("postAnnotation");
      console.log(annotation);
      try {
        const res = await axios.post("/annotations/demo/", annotation);
        return res;
      } catch (err) {
        // Handle Error Here
        console.error(err);
      }
    },
    async putAnnotation(annotation, id) {
      console.log("putAnnotation");
      console.log(annotation);
      try {
        const res = await axios.put("/annotations/demo/" + id, annotation);
        console.log(res);
      } catch (err) {
        // Handle Error Here
        console.error(err);
      }
    },
    async deleteAnnotation(id) {
      console.log("deleteAnnotation");
      console.log(id);
      try {
        const res = await axios.delete("/annotations/demo/" + id);
        console.log(res);
      } catch (err) {
        // Handle Error Here
        console.error(err);
      }
    },
  },
  mounted() {
    self.anno = new Annotorious({
      image: document.getElementById("desk"),
    });
    self.anno.setAuthInfo({
      displayName: "miiiy.rocks",
    });

    this.getAnnotations();

    self.anno.on("createAnnotation", async (annotation) => {
      // take a copy of original annotation
      let newAnnotation = JSON.parse(JSON.stringify(annotation));
      // can remove original annotation
      self.anno.removeAnnotation(annotation);
      // POST without id field
      delete newAnnotation.id;
      const res = await this.postAnnotation(newAnnotation);
      // use the response as our annotation
      self.anno.addAnnotation(res.data);
    });

    self.anno.on("deleteAnnotation", async (annotation) => {
      let id = annotation.id.split("/").pop();
      this.deleteAnnotation(id);
    });

    self.anno.on("updateAnnotation", async (annotation) => {
      let id = annotation.id.split("/").pop();
      this.putAnnotation(annotation, id);
    });
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.container {
  max-width: 800px;
  margin: 30px auto;
  overflow: auto;
  min-height: 300px;
}

.btn {
  box-shadow: inset 0px 1px 0px 0px #ffffff;
  background: linear-gradient(to bottom, #ffffff 5%, #f6f6f6 100%);
  background-color: #ffffff;
  border-radius: 6px;
  border: 1px solid #dcdcdc;
  display: inline-block;
  cursor: pointer;
  color: #666666;
  font-family: Arial;
  font-size: 15px;
  font-weight: bold;
  padding: 6px 24px;
  text-decoration: none;
  text-shadow: 0px 1px 0px #ffffff;
}
.btn:hover {
  background: linear-gradient(to bottom, #f6f6f6 5%, #ffffff 100%);
  background-color: #f6f6f6;
}
.btn:active {
  position: relative;
  top: 1px;
}
</style>
