# servo-rancher
Optune servo driver for Rancher (v1.6)

This driver supports an application deployed via Rancher as a Stack.

A Rancher Stack may contain multiple containers, each of which may individually be modified by the adjust driver.
Each container in the Stack is considered a separately adjustable component.

For each component of the application, the following settings are automatically available when 'adjust --info' is run:
* scale (number of container instances)
* memory reservation
* milli-cpu reservation
* environment variables

A configuration file must be present to define scale of variable modification config.yaml, any custom component settings in it are also returned by 'adjust --info' and map to environment variables of the matching container.

Currently the following parameters are supported:

environment variables:
* MEMORY: defines the java heap size in Kilobytes

"Docker Compose" variables:
* cpu-reservation: millicpu share of the CPU resources
* memory-reservation: memory reserved reserved in bytes/1024 (scale parameter will adjust)

The following is an example config.yaml document, with examples of the available settings:
```
## These elements are required
# Config group for rancher based adjust driver
rancher:
# Rancher Environment(UI) or Project(API) name
  project: "Default"
# Target Rancher Stack name
  stack: "http-test"
## These elements are optional
# API key - if not passed as a secret via rancher (api_key)
  api_key: "ABCDEFG"
# API secret key - if not passed as a secret via rancher (api_secret)
  api_secret: "HIJKLMNO"
# Default parameters for any otherwise undefined resource
  default:
    environment:
      MEMORY:
        min: 4096
        max: 16384
        step: 64
# Template parameter to define transformation needed between Optune and end device
        template: "{{name|toupper}}: '{{parameter}}M'"
# Docker standard parameters would be set in the "compose" section
    compose:
# Options are cpu, memory, and replicas
      cpu:
        min: 500
# Service section for specific service targets/hints
  service:
# Name of service is also the group key
    front:
# Parameter section to modify, allowed values are 'environment' and 'compose'
      environment:
# Example to highlight another templated environment variable
        GC:
          template: "{{name}}: '-XX:+UsePArallelGC -XX:ParallelGCThreads={{parameter}}'"
# Services to exclude
    http-slb:
      exclude: true
# Alternate exclusion form (no service section)
  exclude:
    - http-slb
```

NOTE: MEMORY environment variable defines the java Memory allocation, and will inform the minimum memory available in the reservation parameter if both are present by including a 20% overhead above the MEMORY parameter for the memory-reservation.

Limitations:
- works on default "docker compose" compute and memory parameters
- it is currently possible to pass one environment variable: memory
- GARBAGECOLLECTOR variable is not available, but will initially only support change in type, not internal values

# GET GOING
```
python3 -m venv .venv
source .venv/bin/activate
pip install -r ./requirements.txt

./basic.py
