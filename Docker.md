Docker
======

_Work in progress_

https://docs.docker.com/

- TODO: list of components

https://docs.docker.com/installation/debian/


    sudo docker run --rm hello-world

- TODO: What does that `--rm` mean?


Docker Registry
---------------

- Previous, deprecated version: https://github.com/docker/docker-registry
- Current version: https://github.com/docker/distribution
  - although the docker image is called `registry:2`

Some links:  

- [Docker docs: Docker Registry 2.0](https://docs.docker.com/registry/)
- [Docker docs: Understanding the Registry](https://docs.docker.com/registry/introduction/)
- [Docker docs: Deploying a registry server](https://docs.docker.com/registry/deploying/)
- [Docker docs: Authenticating proxy with nginx](https://docs.docker.com/registry/nginx/)
- [docs/configuration.md](https://github.com/docker/distribution/blob/master/docs/configuration.md)
- [docs/deploying.md](https://github.com/docker/distribution/blob/master/docs/deploying.md)

### How to run private docker repository

We will use S3 as a storage. We will not setup SSL here - use nginx as a reverse proxy. But it is possible to configure registry with SSL (recommended if you are not using SSL-terminating proxy such as nginx).

Create AWS IAM user and attach S3 policy such as [this]( https://github.com/messa/tips/blob/master/AWS.md#s3-iam-policy-for-readwrite-access-only-to-a-single-bucket).

Setup directory structure:

- `/srv/docker-registry/` (for example)
  - `auth/`
    - `htpasswd` (see instructions below)
  - `config.yml` (see example below)
  - `data/` (empty directory)

Prepare `auth/htpasswd` with Registry user credentials:

    mkdir -p auth
    docker run --entrypoint htpasswd registry:2 -Bbn testuser testpassword > auth/htpasswd

For example, contents of `auth/htpasswd` should look like this:

    messa:$2y$05$D8GkaaaS.w2G2Xd6aaFsp.C2fdRaa3sb.e9gU2fdRrAf.ZFVawC7.

Create `config.yml`, for example:

    version: 0.1
    storage:
      s3:
        accesskey: AKI...EUA
        secretkey: 1YXS...6um
        region: eu-central-1 
        bucket: my-docker-registry
        v4auth: true
        rootdirectory: /docker-registry
    auth:
      htpasswd:
        realm: Leadhub Docker Registry
        path: /auth/htpasswd
    http:
      addr: 0.0.0.0:5000
      secret: 3o3...269e

You can generate the `http.secret` for example with `openssl rand -base64 33`.

If you are curious, this is example of the directory structure that Registry creates in S3 bucket:

    $ aws s3 ls --region eu-central-1 --recursive  $bucket_name
    2015-10-13 12:30:32   65789689 docker-registry/docker/registry/v2/blobs/sha256/01/012a7829fd3ffd2165e76e721ac5384131de4ee35e5b34330f5df9d4f52935d6/data
    2015-10-13 12:24:38      71472 docker-registry/docker/registry/v2/blobs/sha256/41/41158247dd502205fcfc5201164f687b957a76d32726e7b45185f22b62143e94/data
    2015-10-13 12:24:36        682 docker-registry/docker/registry/v2/blobs/sha256/91/916b974d99af866381ea9e3c929b4709058946bb44f3ad10dacfc6ea3b2a936b/data
    2015-10-13 12:24:34         32 docker-registry/docker/registry/v2/blobs/sha256/a3/a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4/data
    2015-10-13 12:30:43        437 docker-registry/docker/registry/v2/blobs/sha256/b6/b66fc3a0052fceecab096b5af3c1953a5cb37fce20c9e5557f48495950d7b433/data
    2015-10-13 12:30:35       9765 docker-registry/docker/registry/v2/blobs/sha256/c4/c40e2a3bb47f5f0be021f0a3469cb1cda2251ccfc0ee674eed7629f897270f25/data
    2015-10-13 12:30:36        437 docker-registry/docker/registry/v2/blobs/sha256/fe/fe946d4e45f59fa276e02b8c322eeb135d9987773182a47566f11466fb390a05/data
    2015-10-13 12:30:34         71 docker-registry/docker/registry/v2/repositories/ubuntu/_layers/sha256/012a7829fd3ffd2165e76e721ac5384131de4ee35e5b34330f5df9d4f52935d6/link
    2015-10-13 12:24:39         71 docker-registry/docker/registry/v2/repositories/ubuntu/_layers/sha256/41158247dd502205fcfc5201164f687b957a76d32726e7b45185f22b62143e94/link
    2015-10-13 12:24:37         71 docker-registry/docker/registry/v2/repositories/ubuntu/_layers/sha256/916b974d99af866381ea9e3c929b4709058946bb44f3ad10dacfc6ea3b2a936b/link
    2015-10-13 12:24:35         71 docker-registry/docker/registry/v2/repositories/ubuntu/_layers/sha256/a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8a7c22955b46d4/link
    2015-10-13 12:30:43         71 docker-registry/docker/registry/v2/repositories/ubuntu/_manifests/revisions/sha256/c40e2a3bb47f5f0be021f0a3469cb1cda2251ccfc0ee674eed7629f897270f25/link
    2015-10-13 12:30:43         71 docker-registry/docker/registry/v2/repositories/ubuntu/_manifests/revisions/sha256/c40e2a3bb47f5f0be021f0a3469cb1cda2251ccfc0ee674eed7629f897270f25/signatures/sha256/b66fc3a0052fceecab096b5af3c1953a5cb37fce20c9e5557f48495950d7b433/link
    2015-10-13 12:30:36         71 docker-registry/docker/registry/v2/repositories/ubuntu/_manifests/revisions/sha256/c40e2a3bb47f5f0be021f0a3469cb1cda2251ccfc0ee674eed7629f897270f25/signatures/sha256/fe946d4e45f59fa276e02b8c322eeb135d9987773182a47566f11466fb390a05/link
    2015-10-13 12:30:43         71 docker-registry/docker/registry/v2/repositories/ubuntu/_manifests/tags/latest/current/link
    2015-10-13 12:30:43         71 docker-registry/docker/registry/v2/repositories/ubuntu/_manifests/tags/latest/index/sha256/c40e2a3bb47f5f0be021f0a3469cb1cda2251ccfc0ee674eed7629f897270f25/link
    2015-10-13 12:25:47         20 docker-registry/docker/registry/v2/repositories/ubuntu/_uploads/7b8e675c-acf6-427e-ad4c-7c29ea4a67d0/startedat
    2015-10-13 12:24:55         20 docker-registry/docker/registry/v2/repositories/ubuntu/_uploads/8ec87454-21ba-4972-9953-758392d1119a/startedat

Now just run it:

    exec docker run \
    	-d -p 5000:5000 --restart=always \
    	--name registry \
      	-v /srv/docker-registry/config.yml:/etc/docker/registry/config.yml \
    	-v /srv/docker-registry/auth:/auth \
    	-v /srv/docker-registry/data:/var/lib/registry \
      	registry:2

Point nginx reverse proxy to port 5000:

    server {
    	listen 80;
    	server_name docker.leadhub.cz;

    	location / {
	    	return  301 https://$server_name$request_uri;
	    }
    }

    server {
	    listen 443 ssl;
	    server_name docker.example.com;

    	ssl_certificate     /etc/nginx/ssl/docker.example.com.cert;
    	ssl_certificate_key /etc/nginx/ssl/docker.example.com.key;
    	ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    	ssl_ciphers         HIGH:!aNULL:!MD5;

        # disable any limits to avoid HTTP 413 for large image uploads
        client_max_body_size 0;

    	# required to avoid HTTP 411: see Issue #1486 (https://github.com/docker/docker/issues/1486)
    	chunked_transfer_encoding on;

    	location = / {
    		add_header Content-Type text/plain;
    		return 200 "Docker Repository";
    	}

    	location /v2/ {
    		# Do not allow connections from docker 1.5 and earlier
    		# docker pre-1.6.0 did not properly set the user agent on ping, catch "Go *" user agents
    		if ($http_user_agent ~ "^(docker\/1\.(3|4|5(?!\.[0-9]-dev))|Go ).*$" ) {
    			return 404;
    		}

    		proxy_pass http://127.0.0.1:5000;
    		proxy_set_header Host              $http_host;
    		proxy_set_header X-Real-IP         $remote_addr;
        	proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
    		proxy_set_header X-Forwarded-Proto $scheme;
    		proxy_read_timeout 900;
    	}
    }

Reload nginx:

    sudo systemctl reload nginx
