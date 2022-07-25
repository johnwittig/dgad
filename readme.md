<div align="center">
<a href="https://gitlab.com/cossas/dgad/-/tree/master"><img src="dgad-logo.jpg"/>


![https://cossas-project.org](https://img.shields.io/badge/website-cossas--project.org-orange)
![Commits](https://gitlab.com/cossas/dgad/-/jobs/artifacts/master/raw/ci_badges/commits.svg?job=badge:commits)
![Pipeline status](https://gitlab.com/cossas/dgad/badges/master/pipeline.svg)
![Version](https://gitlab.com/cossas/dgad/-/jobs/artifacts/master/raw/ci_badges/version.svg?job=badge:version)
![License: MPL2.0](https://gitlab.com/cossas/dgad/-/jobs/artifacts/master/raw/ci_badges/license.svg?job=badge:license)
![Code-style](https://gitlab.com/cossas/dgad/-/jobs/artifacts/master/raw/ci_badges/codestyle.svg?job=badge:codestyle)
</div></a>

<hr style="border:2px solid gray"> </hr>
<div align="center">
Hunt domains generated by Domain Generation Algorithms to identify malware traffic
</div>
<hr style="border:2px solid gray"> </hr>

_All COSSAS projects are hosted on [GitLab](https://gitlab.com/cossas/dgad/) with a push mirror to GitHub. For issues/contributions check [CONTRIBUTING.md](CONTRIBUTING.md)_ 

## What is it?
Domain generation algorithms (DGAs) are typically used by attackers to create fast changing domains for command & control channels.
The DGA detective is able to tell whether a domain is created by such an algorithm or not by using a [Temporal Convolutional Network](https://github.com/philipperemy/keras-tcn). For example, a domain like `wikipedia.com` is not generated by an algorithm, whereas `ksadjfhlasdkjfsakjdf.com` is. 

|  | Domain | Classification|
| ------ | ------ | --- |
|✅ | `wikipedia.org` | OK |
|❌ | `ksadjfhlasdkjfsakjdf.com` | DGA |

More information can be found on [cossas-project.org](https://cossas-project.org/portfolio/dgad/).

## Installation

### Python
To install the DGA Detective locally, we recommend using a virtual environment:

```bash
# recommended: use a virtual environment
python -m venv .venv
source .venv/bin/activate
pip install dgad
```

### Docker
Otherwise, you can pull the docker image:

```bash
docker pull registry.gitlab.com/cossas/dgad:4.0.0
```

## Usage

### CLI

You can use the `dgad` CLI directly (if installed with `pip`) or through docker.

```bash
# list available commands with
$ dgad --help
Usage: dgad [OPTIONS] COMMAND [ARGS]...

  DGA Detective can predict if a domain name has been generated by a Domain
  Generation Algorithm

Options:
  --help  Show this message and exit.

Commands:
  client  classify domains from cli args or csv/jsonl files
  server  deploy a DGA Detective server

# list options with
# dgad client --help
# dgad server --help
```

#### CLI Examples

```
$ dgad client -d kajsdfhasdlkjfh.com
$ docker run -i registry.gitlab.com/cossas/dgad:4.0.0 client -d kdsjhfalksdjf.com
$ dgad client -d dsfjkhalsdkfj.com -o json
$ dgad client -d wikipedia.org -d anotherdomain.com
$ dgad client -f tests/data/domains_todo.csv --format csv
$ cat tests/data/domains_todo.csv | dgad client -fmt csv -f -
$ dgad client -f tests/data/domains_todo.csv --format csv --verbosity DEBUG
```

#### CLI input/output

`dgad` CLI can read domains from txt, csv, and jsonl files.
You can find examples of these files in the `demo` top level directory.
`csv` and `jsonl` files are expected to specify which column contains the domains.
See the option `--domains_column` to set this parameter, which is `domain` by default.

```bash
# you can pipe input data to the flag -f from another command
$ cat tests/data/domains_todo.csv | dgad client -fmt csv -f -

# this is especially  useful when using the cli through docker
$ cat demo/domains.jsonl | docker run -i registry.gitlab.com/cossas/dgad:4.0.0 client -fmt jsonl -f -
{
  "domain": "python.org",
  "is_dga": false
}
(...)

# dgad outputs plain json, so you can easily pipe stdout to another command
$ dgad client -f tests/data/domains_todo.csv -fmt csv | jq '{domain: .[0].raw, is_dga: .[0].is_dga}'
{
  "domain": "wikipedia.org",
  "is_dga": false
}
```


### production deployment with gRPC API

In production you may want to split client and server. DGA Detective ships with a performant gRPC api. You can then scale the amount of servers to handle as many domains as you need.

**Server**

```bash
# see dgad server --help for all options
# run
dgad server
2022-07-24 13:37:12,097 INFO     started dga detective classifier ce8f8efe-8272-44dd-a0be-cc34a0df752b
```

**Client**

```bash
# use the -r flag to achieve remote analysis
# for example you can reach a dgad server instance deployed at https://dgad.mydomain.com
dgad client -r -h dgad.mydomain.com -p 443 -f tests/data/domains_todo.csv -fmt csv | jq -r '.[] | {domain: .raw, is_dga: .is_dga}'
{
  "domain": "klajsdfiuweakjvnzslkvjneaiuvbkjbre.ru",
  "is_dga": true
}
{
  "domain": "aksdjhflkajsdhflka.com",
  "is_dga": true
}
{
  "domain": "wikipedia.org",
  "is_dga": false
}
```

### as a python package in your code

```python
# demo/demo.py
from dgad.prediction import Detective
from dgad.utils import pretty_print

mydomains = ["adslkfjhsakldjfhasdlkf.com"]
detective = Detective()
# convert mydomains strings into dgad.schema.Domain
mydomains, _ = detective.prepare_domains(mydomains)
# classify them
detective.investigate(mydomains)
# view result, drops padded_token_vector for pretty printing
pretty_print(mydomains, output_format="json")
```
```bash
python demo.py
[
  {
    "raw": "adslkfjhsakldjfhasdlkf.com",
    "words": [
      {
        "value": "adslkfjhsakldjfhasdlkf",
        "padded_length": 120,
        "binary_score": 0.992063581943512,
        "binary_label": "DGA",
        "family_score": 0.34756162762641907,
        "family_label": "necurs"
      }
    ],
    "suffix": "com",
    "is_dga": true,
    "family_label": "necurs",
    "padded_length": 120
  }
]
```

## Contributing

Contributions to the DGA Detective are highly appreciated and more than welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md) for more information about our contributions process. 

### Setup development environment
To create a development environment to make a contribution, follow these steps:

**Requirements**
* python >= 3.7
* [poetry](https://python-poetry.org)

**Setup**
```bash
# checkout this repository
git clone git@gitlab.com:cossas/dgad.git
cd dgad

# install project, poetry will spawn a new venv
poetry install

# gRPC code generation
make protoc
```

## About

DGA Detective is developed by TNO in the [SOCCRATES innovation project](https://soccrates.eu). SOCCRATES has received funding from the European Union’s Horizon 2020 Research and Innovation program under Grant Agreement No. 833481.
