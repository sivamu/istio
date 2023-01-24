.PHONY: delete recreate help

help :
	@echo "Usage:"
	@echo "   make help          - prints this msg"
	@echo "   make all           - delete and recreate a new k3d cluster"
	@echo "   make delete        - delete k3d cluster"
	@echo "   make deploy-book-info        - deploy the istio default book-info app"

delete :
	# delete the cluster (if exists)
	@# this will fail harmlessly if the cluster does not exist
	-k3d cluster delete ngsa-k3d-cluster

all : delete
	# build k3d cluster
	@k3d cluster create ngsa-k3d-cluster --registry-use k3d-registry.localhost:5000 --config ./k3d.yaml --k3s-arg "--no-deploy=traefik@server:0"

	# wait for cluster to be ready
	@kubectl wait node --for condition=ready --all --timeout=60s
	@sleep 5
	@kubectl wait pod -A --all --for condition=ready --timeout=60s

	# install istio
	@/usr/local/istio/bin/istioctl install --set profile=default -y -f ./deploy/istio-mesh-config.yaml

deploy-book-info :
	# deploy bookinfo
	@kubectl apply -f ./deploy/book-info/book-info.yaml

	# deploy bookinfo-gateway
	@kubectl apply -f ./deploy/book-info/book-info-gateway.yaml

	# deploy bookinfo-versions
	@kubectl apply -f ./deploy/book-info/book-info-versions.yaml

	# deploy bookinfo-routing
	@kubectl apply -f ./deploy/book-info/book-info-routing.yaml

