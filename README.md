Goals
-----

The primary mission of these utilities is to provide a consistent interface around an ecosystem of infrastructure-as-code tools and environments, and to allow for the rapid creation, testing, and iteration of new ideas.


Terms
-----

To help understand the structure and use of these utilities, it helps to understand some of the vocabulary we use:
  * **universe**: A universe is a collection of all the various resources and theories that comprise a given working environment. This is similar in scope to a "project."
  * **theory**: A theory is a set of configurations and variables that flesh out a universe.
  * **environment**: An environment is a running instance of a theory, inside of its universe.

Abstractly, we may want to create a new environment within which to run a set of QA services. We might use Terraform to create the infrastructure, and Ansible to provision it, all within AWS. However, we may want to create multiple identical environments, in different regions. In this case, we would have a single QA universe, with multiple theories of operation -- one for each region we anticipate running a QA environment within. Of course, theories can vary by more than just region. Each theory can specify its own sets of provider configurations, credentials, etc. Modifying our abstract example, instead of multiple regions within AWS, we could have a theory for AWS, and another for GCP.


Usage
-----

First things first, run `theorycraft new my_universe`. This will create a folder structure and several template files. These inclue the base folder, `my_universe`, and within that folder, a `theories` folder to contain your various operational theories, a default theory, a `.gitignore` file, and empty folders to hold Terraform, Packer, and Ansible documents. From this point on, all other theorycraft operations require that you run theorycraft from the root universe folder.

Before getting too far along, however, it would be a good idea to open up the `default.json` file within the newly created `theories` folder, and have a look around. We set up a good selection of parameters to kick things off, but you may find you don't need some of them. Feel free to just remove any section that doesn't apply to your work, theorycraft will carry on. In fact, you can also discard any folders that theorycraft initialized which you may not have need of, as well!

Once you've configured your default theory, theorycraft can now simplify many routine operational tasks. `theorycraft consul` removes the need to remember myriad command line switches and environment variables to successfully query the keystore, and `theorycraft terraform` means never having to type `-backend-config=` again.

Best of all, you can now switch between projects/environments/clients as quickly as changing directories! 


Advanced
--------

Theorycrafted universes are designed to cooperate with version control systems. You should be able to check your universe into git without fear of compromising security or data, to which extent we've given some consideration to sensitive data such as credentials.

For any theory, theorycraft looks for a matching ".vars" file (ie: if you were running against the default theory, default.json, then theorycraft would also look for default.vars). These .vars files should be excluded from version control, and if you're using the generated .gitignore file, will already be addressed. Inside these .vars files, you can specify ':'-separated fields, where the first field is a variable name, and the second field is the value to use. When theorycraft interpolates the running theory, it will replace every occurrence of each variable within a theory with the value from the vars file, as so:

default.vars:
CONSUL_TOKEN:your-consul-agent-token-here

default.theory:
{
    "consul": {
        "address": "consul.localhost:8500",
        "token": "%CONSUL_TOKEN%"
    }
}

Interpolated output:
{
    "consul": {
        "address": "consul.localhost:8500",
        "token": "your-consul-agent-token-here"
    }
}

Realizing that storing credentials in files *at all* is not the best of possible solutions, theorycraft will also allow in-line self-referencing through `$()` expansion, ie:
{
    "vault": {
        "address": "vault.localhost:8200"
    },
    "consul": {
        "address": "consul.localhost:8500",
        "token": "$(vault get dev/consul-token)"
    }
}

Anything contained within `$()` is assumed to be the result from another theorycraft command within the current theory, but theorycraft doesn't recurse, so any target resources need to be fully configured and not rely on `$()` expansions themselves.


Authors
-------

* M. Isaac Holger <isaac@paralleltheory.com>


Copyright
---------

(c)2019 M. Isaac Holger / Parallel Theory LLC
All rights reserved.

