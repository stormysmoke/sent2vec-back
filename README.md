# Sent2Vec

This is the Sent2Vec back-end. It allows to index the sentences of a text (encoding them as vectors), and then to retrieve the "nearest neighbours" of a query sentence.

## Conceptual Usage Example

~~~
index("This is a sentence. Another sentence. And so on...")
query("That's the query sentence.", 3)
~~~

The second argument of the `query` method is the number of nearest neighbours to return.

The `query` method returns the specified number of nearest neighbour sentences along with their "distances" to the query sentence. The smaller the distances, the more similar are the two sentences in meaning.

## Deployment

The application is provided as the [stormysmoke/sent2vec-back](https://hub.docker.com/r/stormysmoke/sent2vec-back/) Docker image on Docker Hub.

The image versions with a tag ending in `-dev` are for development only. They contain a stripped-down Sent2Vec model that is fast to load and deploy, but doesn't produce meaningful results. The other image versions are production-ready.

### Compute Instance Requirements

The Docker image is currently about 7 GB in size (most of this space is due to the Sent2Vec model). Thus, a computing instance to run the image has the following minimum requirements:

- 14 GB free disk storage (double the size of the image for extracting the image after downloading it)
- 7 GB RAM
- 1 CPU (performance with more CPUs might be better, but I didn't test it)

In general, it works fine on a [t2.large](https://aws.amazon.com/ec2/instance-types/t2/) EC2 instance on AWS.

### Prerequisites

#### 1. RabbitMQ

The application uses RabbitMQ for communicating with the client.

Make sure you have a running RabbitMQ server listening on the following ports:

- 5672 (default)
- 1723

Note the URI of this server. A RabbitMQ server URI has the following format:

~~~
amqp://user:password@host:port/virtualhost
~~~

You will need this information when you start the Docker image of the application.

#### 2. AWS S3

The application uses an AWS S3 bucket as a persistent storage.

Create an S3 bucket and note the following information about it:

- Name of the bucket
- AWS access key of an AWS account that has write access to this bucket
- AWS secret access key of this account

You will need this information when you start the Docker image of the application.

### Deployment

#### 1. Install Docker

Make sure Docker is installed on the computing instance.

To install Docker on Ubuntu:

~~~bash
sudo apt-get update
sudo apt-get -y install docker.io
~~~

#### 2. Run the Docker Image

~~~bash
docker run \
    -d \
    -e RABBITMQ_URI=<uri> \
    -e AWS_ACCESS_KEY_ID=<key> \
    -e AWS_SECRET_ACCESS_KEY=<key> \
    -e S3_BUCKET_NAME=<name> \
    stormysmoke/sent2vec-back:<tag>
~~~

As you can see, you need to pass the information that you noted down in the previous section as environment variables to the Docker image.

### Monitoring 

You can see the output of the application with `docker logs <container>`, where `<container>` is the running container's ID.

## Communication

The application connects to a RabbitMQ server (running on Heroku) and acts as a RPC server. The message format is inspired by JSON-RPC 2.0 (and eventually should be conformant with this standard), and currently looks as follows:

### Method: `index`

Request:

~~~json
{"method": "index", "params": "This is a sentence. And so on..."}
~~~

Response:

~~~json
{"result": null}
~~~

### Method: `query`

Request:

~~~json
{"method": "query", "params": ["That's the query sentence.", 3]}
~~~

Response:

~~~json
{"result": [["Sentence 1", "Sentence 2", "Sentence 3"], [0.84, 0.94, 1.04]]}
~~~
