# .Net Interactive

## Simple start

Try out .NET interactive with the docker image :[link](https://github.com/dotnet/interactive/tree/main/samples/docker-image)

docker build . --tag dotnet-interactive:1.0
docker run --rm -it -p 8888:8888 -p 1000-1200:1000-1200  --name dotnet-interactive-image dotnet-interactive:1.0
Open the page at [http://127.0.0.1:8888/](http://127.0.0.1:8888/)