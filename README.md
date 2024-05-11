#  Auto-Instrument-JavaScript-NodeJS

This project provides a guide how to automatically instrument a NodeJS application with OpenTelemetry. Furthermore, it is possible to connect it with [ExplorViz](https://github.com/explorviz), so the information the traces give about the software is visualized. The NodeJS application I choose is a [MongoDB admin panel](https://github.com/mongo-express/mongo-express/tree/master). The guide is focused on that, but feel free to connect the instrumentation with your own NodeJS project.

## Table of Contents

- [Auto-Instrument-JavaScript-NodeJS](#Auto-Instrument-JavaScript-NodeJS)
  - [Table of Contents](#table-of-contents)
  - [About the Project](#about-the-project)
  - [Getting Started](#getting-started)
    - [Prerequisites](#prerequisites)
    - [Installation](#installation)
  - [License](#license)

## About the Project

This project aims to give an example application for deploying the autoamtic instrumentation by OpenTelemetry for NodeJS applications. It can be used for any project of their own.  

## Getting Started

Instructions for setting up the project on a local machine.

### Prerequisites

The user needs to have both Docker and Docker compose installed.

So first, you need to clone into this repository as well as into the [MongoDB admin panel](https://github.com/mongo-express/mongo-express/tree/master). Both project should be merged, which is not a big deal, since only a README exists in both.

Now you need to install the following dependencies, which are crucial for OpenTelemetry:

yarn add \
  @opentelemetry/api \
  @opentelemetry/auto-instrumentations-node \
  @opentelemetry/exporter-collector \
  @opentelemetry/exporter-metrics-otlp-proto \
  @opentelemetry/exporter-trace-otlp-proto \
  @opentelemetry/node \
  @opentelemetry/sdk-node \
  @opentelemetry/tracing

The next step is to update the Dockerfile, so that the automatic instrumentation, which we set up in tracer.cjs (if you use your own project and .js works, just rename the file), is used when the project is build. Therefore, you have to add 

COPY --from=build /dockerbuild/tracer.cjs /opt/mongo-express/ 

to the COPY commands in the existing Dockerfile and update the CMD command at the very end to 

CMD ["/sbin/tini", "--", "node", "--require", "./tracer.cjs", "app.js"]

Now, I already prepared the docker-compose.yaml, so that the installation of the MongoDB admin panel becomes easier. Just comment out the two containers.

If you plan to use ExplorViz, then please clone into the [Deployment](https://github.com/ExplorViz/deployment) and the [Frontend](https://github.com/ExplorViz/frontend) and follow their instructions to get the software running. 

### Installation

Now, you have two options: Either send the traces and metrics to Zipkin and Prometheus or export the traces to ExplorViz. 

When doing the first, you need to comment out both exporter in the collector-config.yaml and also insert them as the exporter in the OpenTelemetry pipeline. The command "docker-compose up --build -d" builds the project and deploys all containers. Then go to the address in your browser, where the MongoDB admin panel is deployed and explore it. After doing this, you created traces and metrics, which can be checked out by going to [http://localhost:9411](http://localhost:9411) and [http://localhost:9090](http://localhost:9090). 

If you plan on integrating this project with ExplorViz, then you can leave the collector-config.yaml as it is. Just comment out the otlphttp exporter, which would send the metrics to the a metric-service ExplorViz does not have yet. Update in the .env the token and the secret with the values of your ExplorViz landscape and again just build and deploy with "docker-compose up --build -d". 

## License

License can be found [here](/LICENSE)

