# Benchmarking: Installation guide

## Seqera Platform

You will need access to the following within Seqera Platform:

- [Workspace](https://docs.seqera.io/platform/23.3.0/orgs-and-teams/workspace-management)
- Minimum of [Maintain](https://docs.seqera.io/platform/23.3.0/orgs-and-teams/workspace-management#participant-roles) user permissions within the Workspace
- [Compute Environment](https://docs.seqera.io/platform/23.3.0/compute-envs/overview) that has already been pre-configured
- [Access token](https://docs.seqera.io/platform/23.3.0/api/overview#authentication) which is your personal authorization token for the Seqera Platform CLI. This can be created in the user menu under **Your tokens**.


## Software dependencies

### Installing seqerakit

This tutorial requires [`seqerakit >=0.4.3`](https://github.com/seqeralabs/seqera-kit#installation) as well as the following dependencies:

1. [Seqera Platform CLI (`>=0.9.0`)](https://github.com/seqeralabs/tower-cli#1-installation)

2. [Python (`>=3.8`)](https://www.python.org/downloads/)

3. [PyYAML](https://pypi.org/project/PyYAML/)

There are two primary ways to install `seqerakit`:

1. Conda

   seqerakit is available on [Bioconda](https://anaconda.org/bioconda/seqerakit) and this is the easiest method to install all of the software dependencies listed above within a single environment. If you don't have a Conda distribution installed already, [Miniconda](https://docs.conda.io/projects/miniconda/en/latest/miniconda-install.html) is a good, lightweight option that is supported on Windows, macOS, and Linux.

   1. You can create a Conda environment with `seqerakit` and all of it's dependencies from the [`environment.yml`](../environment.yml) provided in the top-level folder using the following command:

      ```bash
      conda env create -f environment.yml
      ```

   2. Once the installation process is complete you can activate and use this Conda environment with the following command:

      ```bash
      conda activate seqerakit
      ```

2. Manual

   If you prefer to install each of the dependencies required by `seqerakit` individually you can manually install:

   1. Seqera Platform CLI

      Once you have completed the [installation](https://github.com/seqeralabs/tower-cli#1-installation), check that the Seqera Platform CLI is installed properly using the following command:

      ```bash
      tw --help
      ```

   2. Python (`>=3.8`)

      Once you have completed the [installation](https://www.python.org/downloads/), check that Python is installed properly using the following command:

      ```bash
      python --version
      ```

   3. PyYAML

      Once you have completed the [installation](https://pypi.org/project/PyYAML/) with the command below:

      ```bash
      pip install PyYAML
      ```

      Check that PyYAML is installed properly using the following command:

      ```bash
      pip show pyyaml
      ```

   4. `seqerakit`

      Once you have completed the [installation](https://pypi.org/project/seqerakit/) with the command below:

      ```bash
      pip install seqerakit
      ```

      Check that `seqerakit` is installed properly using the following command:

      ```bash
      seqerakit --help
      ```

   5. `md5sum`

      Most Linux based operating systems come with `md5sum` pre-installed with the GNU coreutils package. If you are using a Apple Mac, you can install it using [homebrew for mac](https://brew.sh/).

      Check that `md5sum` is installed properly using the following command:

      ```bash
      md5sum --help
      ```

## Post-installation

Once you have successfully installed `seqerakit` you will need to check that both the Seqera Platform CLI and `seqerakit` have been configured appropriately to access the Seqera Platform via the command-line.

If you are unsure as to which of these settings you need for your particular Seqera Platform installation, please refer to the options in the [Support](#support) section.

### Access token (All customers)

If you haven't already, you will need to create an access token from the user interface via the **Your Tokens** page in your profile.

You will then need to make the access token available to the Seqera Platform CLI and `seqerakit`. You can do this by either:

1. Exporting the token as a shell variable directly into your terminal:

   ```bash
   export TOWER_ACCESS_TOKEN=<your access token>
   ```

2. Adding the above `export` command to a file such as `.bashrc` to be automatically added into your environment (recommended).

### API endpoint (Enterprise customers only)

For Enterprise installations of Seqera Platform, you will also need to configure the API endpoint that will be used to connect to the Platform. You can do this by either:

1. Exporting the token as a shell variable directly into your terminal:

   ```bash
   export TOWER_API_ENDPOINT=<your API endpoint>
   ```

2. Adding the above `export` command to a file such as `.bashrc` to be automatically added into your environment (recommended).

Seqera Cloud customers won't need to configure the API endpoint as the default value is appropriate.

### TLS certificates (Enterprise customers only)

For Enterprise installations of Seqera Platform which do not present a TLS certificate, you may need to qualify and run Seqera Platform CLI commands with the `--insecure` parameter:

```bash
tw --insecure info
```

When using `seqerakit`, this setting can be similarly used with the `--cli=` flag provided to your `seqerakit` command:

```bash
seqerakit <yaml file> --cli="--insecure"
```

Seqera Cloud customers won't need to configure this setting as the default value is appropriate.

### Custom SSL certificates (Enterprise customers only)

For Enterprise installations of Seqera Platform, if you are using an SSL certificate that it is not accepted by the default Java certificate authorities you can [customize](https://www.baeldung.com/jvm-certificate-store-errors) a `cacerts` store and use it like when invoking the Seqera Platform CLI:

```bash
tw -Djavax.net.ssl.trustStore=/absolute/path/to/cacerts info
```

You can similarly specify the path to a custom `cacerts` store to the `seqerakit` command using the `--cli` flag:

```bash
seqerakit <yaml file> --cli="-Djavax.net.ssl.trustStore=/absolute/path/to/cacerts"
```

Please see the `seqerakit` [documentation](https://github.com/seqeralabs/seqera-kit#using-tw-specific-cli-options) for more information about specifying `tw` CLI options when running `seqerakit`.

Alternatively, to avoid typing it everytime we recommend that you rename the `tw` binary to `tw-binary` and create an executable script called `tw` like below:

```bash
#!/usr/bin/env bash
tw-binary -Djavax.net.ssl.trustStore=/absolute/path/to/cacerts $@
```

Seqera Cloud customers won't need to configure this setting as the default value is appropriate.

## Health check

### Seqera Platform CLI

Once you have applied the appropiate settings to use the Seqera Platform CLI you can confirm the installation, configuration and connection is working as expected with the command below:

```console
$ seqerakit --info

DEBUG:root: Running command: tw info
Details
    -------------------------+----------------------
     Tower API endpoint      | https://api.cloud.seqera.io
     Tower API version       | 1.23.0
     Tower version           | 23.3.0-cycle23
     CLI version             | 0.9.0 (0cea70d)
     CLI minimum API version | 1.15
     Authenticated user      | joebloggs

    System health status
    ---------------------------------------+----
     Remote API server connection check    | OK
     Tower API version check               | OK
     Authentication API credential's token | OK
```

As highlighted in the section above, if you are using an Enterprise installation of Seqera Platform you may also need to customise the Seqera Platform CLI command invocation with one or more of the following settings:

- [API endpoint](#api-endpoint)
- [TLS certificates](#tls-certificates)
- [Custom SSL certificates](#custom-ssl-certificates)

If you are still experiencing issues please refer to the options in the [Support](#support) section.

### seqerakit

The [`TOWER_ACCESS_TOKEN`](#access-token) and [`TOWER_API_ENDPOINT`](#api-endpoint) environment variables will automatically be used by `seqerakit` if you have injected them into the executing environment.

`seqerakit` will support all Seqera Platform specific CLI settings, as shown in [TLS Certificates](#tls-certificates) and [Custom SSL certificates](#custom-ssl-certificates) with use of the `--cli=` flag, except for the `--verbose` or `-v` setting.

For a full list of Seqera Platform CLI options, you can run `tw -h`:

```console
$ tw -h
Usage: tw [OPTIONS] [COMMAND]

Nextflow Tower CLI.

Options:
  -t, --access-token=<token>   Tower personal access token (TOWER_ACCESS_TOKEN).
  -u, --url=<url>              Tower server API endpoint URL (TOWER_API_ENDPOINT) [default: 'tower.nf'].
  -o, --output=<output>        Show output in defined format (only the 'json' option is available at the moment).
  -v, --verbose                Show HTTP request/response logs at stderr.
      --insecure               Explicitly allow to connect to a non-SSL secured Tower server (this is not recommended).
  -h, --help                   Show this help message and exit.
  -V, --version                Print version information and exit.
```

For more information on running seqerakit, please see the `seqerakit` [configuration](https://github.com/seqeralabs/seqera-kit/tree/main#configuration) documentation.

## Support

For Enterprise installations, if you are unsure whether these settings are required when using the Seqera Platform CLI, you have the following options:

- Contact the Administrator of your Seqera Platform instance
- Create a ticket on the Seqera customer support portal
- Contact us at [support@seqera.io](mailto:support@seqera.io)
