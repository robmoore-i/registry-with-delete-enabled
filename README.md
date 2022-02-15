# registry-with-delete-enabled:2

You may have noticed that the docker registry created by the official registry image is actually not CRUDable. It is merely CRUable, because you can't delete image tags from it by default.

This is problematic for when you're using k3d, where custom registry configuration is a complicated nightmare.

Fortunately, I got you.

```
# Start it
$ docker run -d -p 5000:5000 --restart=always --name registry registry-with-delete-enabled:2

# Find some small, random image you have lying around
$ docker image ls # I found rancher:k3d-tools:5.0.2. It's 18MB.

# Tag and push to the local registry
$ docker tag rancher/k3d-tools:5.0.2 localhost:5000/k3d-tools:5.0.2
$ docker push localhost:5000/k3d-tools:5.0.2

# Get digest for deletion...
$ curl -v -H "Accept: application/vnd.docker.distribution.manifest.v2+json" localhost:5000/v2/k3d-tools/manifests/5.0.2
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 5000 (#0)
> GET /v2/k3d-tools/manifests/5.0.2 HTTP/1.1
> Host: localhost:5000
> User-Agent: curl/7.64.1
> Accept: application/vnd.docker.distribution.manifest.v2+json
> 
< HTTP/1.1 200 OK
< Content-Length: 1157
< Content-Type: application/vnd.docker.distribution.manifest.v2+json
< Docker-Content-Digest: sha256:961ce0daa42bd30c8c81642f7ffa62efcbc2bb4ae3a5955fb6e023ced3d246c2 # <--- We need this to delete the image.
< Docker-Distribution-Api-Version: registry/2.0
< Etag: "sha256:961ce0daa42bd30c8c81642f7ffa62efcbc2bb4ae3a5955fb6e023ced3d246c2"
< X-Content-Type-Options: nosniff
< Date: Mon, 17 Jan 2022 05:57:03 GMT
< 
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
   "config": {
      "mediaType": "application/vnd.docker.container.image.v1+json",
      "size": 3469,
      "digest": "sha256:21d320c0abde528892414d1717faac9c4237b5b5a2d50742ee907475c1449cbe"
   },
   "layers": [
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 2814079,
         "digest": "sha256:4e9f2cdf438714c2c4533e28c6c41a89cc6c1b46cf77e54c488db30ca4f5b6f3"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 2802393,
         "digest": "sha256:c1d5d5b9afad453e7c392eb9e145784e2e90ff39e9558807daf578e21fc93fdd"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 126,
         "digest": "sha256:7d21c407c4b981603c1bd18c153f0fc4c6166a19bd7aa01bcc709d3ca740143a"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 3209136,
         "digest": "sha256:a3220e8418833e4593157b3c2c362270d20db12c981a29d00a85b75cb00d3463"
      }
   ]
* Connection #0 to host localhost left intact
}* Closing connection 0

# Delete image
$ curl -X DELETE localhost:5000/v2/k3d-tools/manifests/sha256:961ce0daa42bd30c8c81642f7ffa62efcbc2bb4ae3a5955fb6e023ced3d246c2

# Satisfy outselves that it is really gone
$ curl localhost:5000/v2/k3d-tools/tags/list
{"name":"k3d-tools","tags":null}

### Another epic win for Docker! ###
```
