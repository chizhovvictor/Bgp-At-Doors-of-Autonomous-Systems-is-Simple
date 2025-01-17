1 Install gns3

2 Install docker container 1

docker build -f _vchizhov-1 -t busybox .
docker run -dt --name _vchizhov-1 busybox

3 Install docker container 2

docker build -f _vchizhov-2 -t quagga .
docker run -dt --name _vchizhov-2 .


