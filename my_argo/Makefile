NS = myargo
.PHONY: init
init:
	kubectl apply -f cluster_role.yaml
	kubectl apply -f custom_resources.yaml
.PHONY: install
install:
	helm upgrade --install my-argo --namespace=${NS} .
