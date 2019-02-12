build:
	protoc -I . --go_out=plugins=micro:/root/go/src/shippy/user-service proto/user/user.proto
	GOOS=linux GOARCH=amd64 go build
	docker build --rm -t user-service .

run:
#	docker run -p 50053:50051 -e MICRO_SERVER_ADDRESS=:50051 -e MICRO_REGISTRY=mdns -e       DB_NAME: "postgres" -e DB_HOST: "database" -e DB_PORT: "5432" -e DB_USER: "postgres" -e DB_PASSWORD: "postgres"  user-service

	docker run --rm --net="host" -p 5054 \
                    -e MICRO_REGISTRY=mdns  \
                    -e DB_NAME="postgres"  \
                    -e DB_HOST="127.0.0.1"  \
                    -e DB_PORT="5433"  \
                    -e DB_USER="postgres"  \
                    -e DB_PASSWORD="postgres"  \
                    -e MICRO_BROKER=nats \
                    -e MICRO_BROKER_ADDRESS=0.0.0.0:4222 \
                    --name user-service \
                    user-service
