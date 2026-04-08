# Asset Validator Entrypoint Plugins

## PLC-L1: Cloud Services PLC-L1: Software Architecture and Design Document

**Approval Due Date: Feb 3, 2026**

Documentation Control

| Item | Description |
| :---- | :---- |
| Title | Asset Validator Entrypoint Plugins |
| Author(s) | [Matt Kuruc US](mailto:mkuruc@nvidia.com) |
| Revision | Jan 22, 2026 |
| **State** | Released for Approval |
| PLC-L1 SADD Template Revision | [SWE-PLC-L1-102-CloudPLC-SADD-TMPL, Oct.5, 2023](https://docs.google.com/document/d/1NcBdK9eat4dzu6peyTryREMGy7gyNg2aAw-2rqEGPwo/edit?tab=t.0#heading=h.ihv636) |
| **Associated SRD/PRD(s)** | [26Q1 - GTC - SRD - Content Pipeline Eng Teams](https://docs.google.com/document/d/1LoC-i5CTaqk4ANtOXLBAdjKP6sD_KGS4NjsOg-WYqRE/edit?tab=t.sxdgpu9l9jx7)[Unified Asset Validation Analysis](https://docs.google.com/document/u/0/d/1nZ26D0G5tlHAwVz_W1ZpB9W-yNLYVViQZ242cJB7tKM/edit) |

Approvers

| Date | Name | Approval State | Notes |
| :---- | :---- | :---- | :---- |
| Feb 3, 2026 | [Miguel Angel Hernandez Sanchez DE](mailto:miguelh@nvidia.com) | Approved | Approved. |
| Date | [Ronnie Sharif US](mailto:zsharif@nvidia.com) | Not started |  |
| Mar 5, 2026 | [Kim Dieser US](mailto:kdieser@nvidia.com) | Approved | 3/5 \- revised language in security design section |
| Date | [Jatin Ghori IN](mailto:jghori@nvidia.com) | Not started |  |

Reviewers

| Date | Name | Notes |
| :---- | :---- | :---- |
| Date | [Tae Kim US](mailto:taeyongk@nvidia.com) |  |
| Date | [Dirk Van Gelder US](mailto:dvangelder@nvidia.com) | \<Brief description.\> |
|  | [Bryan Smith AU](mailto:bryans@nvidia.com) |  |
|  | [Martin Watt US](mailto:mwatt@nvidia.com) |  |
| Jan 27, 2026 | [Andrew Kaufman CA](mailto:akaufman@nvidia.com) | Looking good to me. I probably don’t need the on\_shutdown mechanism for USDEX / Converter purposes, but I get why it might be needed for more interactive use cases |
| Jan 30, 2026 | [Jens Jebens AU](mailto:jjebens@nvidia.com) | A bit more detail on documentation would be great (see comments). Other than that it LGTM |
|  | [Kevin Collins US](mailto:kecollins@nvidia.com) |  |
|  | [Jason Batchkoff US](mailto:jbatchkoff@nvidia.com) | SimReady Engineering Lead |
|  | [Jason Wylie CA](mailto:jwylie@nvidia.com) |  |
| 2026-02-03 | [Aaron Luk US](mailto:aluk@nvidia.com) | Reviewed |
|  | [Alexandre Abreu CA](mailto:aabreu@nvidia.com) |  |
|  | [Christian Akesson US](mailto:cakesson@nvidia.com) | VFI and DSX Develop samples of content and validation pipeline. |

Revision History

| Version | Date | Author | Summary of Change |
| :---: | :---- | :---- | :---- |
| 0.1 | Jan 27, 2026 | Matt Kuruc | Initial Draft |
| 0.2 | Feb 3, 2026 | Matt Kuruc | Revised example entrypoint to show explicit registration and deregistration of rules. |
| 0.3 | Mar 5, 2026 | Matt Kuruc | Minor revisions |
| 0.4 | Mar 6, 2026 | Matt Kuruc | Accepted modified version of [Kim Dieser US](mailto:kdieser@nvidia.com)’s recommendations |

Table of Contents

[**Introduction**](#introduction)	**[4](#introduction)**

[Purpose and Scope](#purpose-and-scope)	[4](#purpose-and-scope)

[Assumptions](#assumptions)	[4](#assumptions)

[Constraints](#constraints)	[4](#constraints)

[Dependencies](#dependencies)	[4](#dependencies)

[Definitions, Acronyms, Abbreviations](#definitions,-acronyms,-abbreviations)	[4](#definitions,-acronyms,-abbreviations)

[References](#references)	[4](#references)

[**Architectural Details**](#architectural-details)	**[4](#architectural-details)**

[**Design Details**](#design-details)	**[5](#design-details)**

[Design Alternatives](#design-alternatives)	[5](#design-alternatives)

[Static Design](#static-design)	[5](#static-design)

[Configuration Data](#configuration-data)	[5](#configuration-data)

[External Interface and Specification](#external-interface-and-specification)	[5](#external-interface-and-specification)

[Dependencies](#dependencies-1)	[6](#dependencies-1)

[Integration Validation Plan](#integration-validation-plan)	[6](#integration-validation-plan)

[Dynamic Design](#dynamic-design)	[7](#dynamic-design)

[Functionality and Behavior](#functionality-and-behavior)	[7](#functionality-and-behavior)

[Control Flow](#control-flow)	[7](#control-flow)

[Data Flow](#data-flow)	[7](#data-flow)

[Error Handling](#error-handling)	[7](#error-handling)

[Logging and Debugging](#logging-and-debugging)	[7](#logging-and-debugging)

[State Machine](#state-machine)	[7](#state-machine)

[Security Design](#security-design)	[7](#security-design)

[Test Automation](#test-automation)	[7](#test-automation)

[Other Design Considerations](#other-design-considerations)	[8](#other-design-considerations)

[Resource Limits](#resource-limits)	[8](#resource-limits)

[High Availability](#heading=h.147n2zr)	[9](#heading=h.147n2zr)

[Scalability](#heading=h.3o7alnk)	[9](#heading=h.3o7alnk)

[Future Work](#future-work)	[9](#future-work)

1. # Introduction {#introduction}

   1. ## Purpose and Scope {#purpose-and-scope}

This document designs the interface for registering Omniverse Asset Validator Plugins using Python entrypoints. Python entrypoints provided a standard way for packages to declare themselves as discoverable plugins for another package.

The Omniverse Asset Validator can use this to support extensible requirements, checkers, features, or profiles defined by efforts like SimReady.

The overall design is symmetrical with how Kit extension startup/shutdown works to offer a seamless transition from one paradigm to another.

2. ## Assumptions {#assumptions}

* Omniverse Asset Validator’s primary interface is python  
* pip, uv, poetry, venv and other standard Python packaging tooling have sufficient adoption by users of the asset validator  
* Asset Validator can “dogfood” this design by migrating all core validators to be discoverable via entrypoints without introducing new packages

  3. ## Constraints {#constraints}

* Avoid impact on existing usage for Content Ingestion Pipeline, Exchange SDK, and Omniverse Kit

  4. ## Dependencies {#dependencies}

| Team or Company | Deliverable |
| :---- | :---- |
|  | N/A |

  5. ## Definitions, Acronyms, Abbreviations {#definitions,-acronyms,-abbreviations}

| Term or Abbreviation | Description |
| :---- | :---- |
| Entrypoint | A specification for identifying package provided plugins accessible via importlib: [Entry points specification \- Python Packaging User Guide](https://packaging.python.org/en/latest/specifications/entry-points/#entry-points-specification)  |
| Requirements | An atomically defined and validatable set of conditions |
| Features | A harmonization of requirements that when valid describes a behavior |
| Profile | A high level set of features that a user targets for their asset (e.g., simready-mpv-1.0) |
| SimReady | Set of specifications and tooling to make OpenUSD assets ready for simulation |

  6. ## References {#references}

| Input Work Product | Revision | Location |
| :---- | :---- | :---- |
| Validation Profiles Library Design Questions / Problems | N/A | \<[link](https://docs.google.com/document/d/1M_0BXp4ULwEPITPXLJxj8rsUQxedvSrAC2V-d40Uu9E/edit?tab=t.0)\> |
| Requirement Based Validation | N/A | \<[link](https://docs.google.com/presentation/d/1e7AifSK1ERrv-qkf04cJYUgwkXll_dDm/edit)\> |
| Asset Validator OpenUSD Profiles | 1.10.12 | \<[link](http://omniverse-docs.s3-website-us-east-1.amazonaws.com/usd_profiles/1.10.12/docs/requirements.html)\> |
| Entrypoint Specification |  | \<[link](https://packaging.python.org/en/latest/specifications/entry-points/)\> |
| Writing your Pyproject.toml |  | \<[link](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/#advanced-plugins)\> |
| Binary distribution format \- Python Packaging User Guide |  | \<[link](https://packaging.python.org/en/latest/specifications/binary-distribution-format/#the-data-directory)\> |

2. # Architectural Details {#architectural-details}

**Provide a high-level description of the architecture and include block diagram(s) if applicable**  
The Omniverse Asset Validator will use importlib to run any startup function specified via the “omni.asset\_validator:registrant” entrypoint.  An entrypoint is commonly specified in a project’s pyproject.toml.

```
[project.entry-points."omni.asset_validator"]
registrant = "some_rules:Entrypoint"
```

Python wheel builders will ingest this and produce an “entry\_points.txt”.

```
[omni.asset.validator]
registrant = "some_rules:Entrypoint"
```

The entrypoint for plugins can be identified with this python function.

```py
importlib.metadata.entry_points(group='omni.asset_validator', name='registrant')
```

Entrypoint modules may take on any number of user defined structures including

* Distinct modules for requirements, features, profiles, and validators  
* Uber-modules that register everything

Entrypoint modules are expected to properly declare their dependencies and must not assume any particular loading order.

**Describe the static and dynamic aspects of the architecture**

Entrypoint declaration (such as the entry\_point.txt in a package’s wheel) is the static element of the architecture.

The dynamic aspect of the architecture consists of loading the modules specified in the entrypoint declaration via importlib.

**Describe key assumptions and limitations (if any) of the architecture**  
This does not explicitly address registering of validators produced via the Pixar Animation Studios C++ Validation framework.  Lofting of C++ validators into the OAV framework is a separate feature that should align with this proposal.

3. # Design Details {#design-details}

   1. ## Design Alternatives {#design-alternatives}

Asset Validator is currently extensible via Kit Extensions.  This is rejected as it inserts Kit as a dependency for specifications.

Another discussed approach was to make the entrypoint return explicit lists of checkers, profiles, etc. and let the framework handle loading and unloading. This would diverge too significantly from the current decorator based registry, but may be considered in a future major version redesign.

2. ## Static Design {#static-design}

   1. ### Configuration Data {#configuration-data}

The entry\_points.txt generated from a pyproject.toml included in a python package functions declares the modules that should be loaded.

Users may use any number of formats (requirements.txt, pyproject.toml, etc.) compatible with their package management tooling to specify the packages they depend on.

2. ### External Interface and Specification {#external-interface-and-specification}

   1. #### entry\_points.txt

* [Entry points specification \- Python Packaging User Guide](https://packaging.python.org/en/latest/specifications/entry-points/)  
* [Writing your pyproject.toml \- Python Packaging User Guide](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/#advanced-plugins)

  3. ### Dependencies {#dependencies-1}

| Team or Company | Deliverable | Ref | Commit? |
| :---- | :---- | :---- | :---- |
| Content Ingestion Pipeline | CIP works with entrypoint registration | [Asset Validator Entrypoint Plugins](https://docs.google.com/document/d/1irYm-jpRL7AgiEw2Hk3RUK9aARK0tr73b_SSakDTaIM/edit?tab=t.0#bookmark=id.r49j7od8uane) |  |
| Exchange SDK | Exchange SDK works with entrypoint registration | [Asset Validator Entrypoint Plugins](https://docs.google.com/document/d/1irYm-jpRL7AgiEw2Hk3RUK9aARK0tr73b_SSakDTaIM/edit?tab=t.0#bookmark=id.r49j7od8uane) | Yes |
| CAD Converters | CAD Converters work with entrypoint registration | [Asset Validator Entrypoint Plugins](https://docs.google.com/document/d/1irYm-jpRL7AgiEw2Hk3RUK9aARK0tr73b_SSakDTaIM/edit?tab=t.0#bookmark=id.r49j7od8uane) | Yes |
| Scene Optimizers | The Scene Optimizers provides a standalone library that registers their validators using entrypoints | [Asset Validator Entrypoint Plugins](https://docs.google.com/document/d/1irYm-jpRL7AgiEw2Hk3RUK9aARK0tr73b_SSakDTaIM/edit?tab=t.0#bookmark=id.r49j7od8uane) |  |

     1. #### Integration Validation Plan {#integration-validation-plan}

| Functionality to Validate | Teams Participating | Interfaces Covered | Date Complete |
| :---- | :---- | :---- | :---- |
| To validate the interface, can existing validators move to being ingested as entrypoints? | Asset Validator, CIP, Exchange SDK, Scene Optimizers | Validator Registration |  |
| Does this updated design require a new major version of asset validator? | Asset Validator, CIP, Exchange SDK | User InterfaceCommand Line |  |

  3. ## Dynamic Design {#dynamic-design}

     1. ### Functionality and Behavior {#functionality-and-behavior}

A plugin for the asset validator may have the following structure (or a subset of the following structure, allowing profile or requirement only modules if desired).

```
|- some_rules
  |- pyproject.toml
  |- __init__.py
  |- validators
    |- __init__.py
    |- some_checkers.py
    |- some_more_checkers.py
  |- requirements
    |- __init__.py
    |- some_requirements.py
    |- some_more_requirements.py
  |- features
    |- __init__.py
    |- some_features.py
    |- some_more_features.py
  |- profiles
    |- __init__.py
    |- some_profiles.py
    |- some_more_profiles.py
```

The RegistryEntrypoint is responsible for specifying the on\_startup and on\_shutdown callbacks which should provide symmetry Kit Extension based registration.  

```py
import some_rules.validators.some_checkers
import some_rules.validators.some_more_checkers
import some_rules.requirements.some_requirements
import some_rules.requirements.some_more_requirements

class RegistryEntrypoint:
    @classmethod
    def on_startup(cls):
       """If registering via decorator, it's encouraged that importing modules
       occurs in the startup method. Note that registration via decorator may       fall out of favor in entrypoint based design."""
       register_rule(some_rules.validators.some_checkers.CheckMaterial)
       register_rule(some_rules.validators.some_more_checkers.CheckSize)

    @classmethod
    def on_shutdown(cls):
        """optionally deregister any rules, requirements, etc."""
        deregister_rule(some_rules.validators.some_more_checkers.CheckSize)
	 deregister_rule(some_rules.validators.some_more_checkers.CheckMaterial)
```

The entrypoint is shown here as a class with classmethods, but it’s acceptable for it to be a module or an instance of an object, as long as the on\_startup and on\_shutdown interfaces are properly specified.

```
[omni.asset_validator]
registrant = "some_rules:Entrypoint"
```

2. ### Control Flow {#control-flow}

Users use any standard python package management tooling (poetry, uv, pip) to configure their environment.

On asset validator startup…

* Use importlib to gather all modules (unsorted) registered as “omni.asset\_validator” entrypoints named “registrant”  
* Provide a topological sort based on package dependencies  
* Use importlib to load the “registrant” entrypoint  
* Call “registrant.on\_startup()”  
* If in a Kit context, Kit extensions registration should happen after all entrypoints are loaded.

On asset\_validator shutdown…

* If in a Kit context, Kit extension deregistration should happen before all entrypoints are unloaded  
* Use importlib to gather all modules (unsorted) registered as “omni.asset\_validator” entrypoints named “registrant”  
* Provide a topological sort based on package dependencies  
* Use importlib to load the “registrant” entrypoint  
* Call the “registrant.on\_shutdown()”

The topological sort is designed to allow for packages that “deregister” checkers because they have an optimized or specialized version of a checker. This ensures that the upstream rule has been loaded before the specialized version tries to unload it. Sort order outside of direct dependencies should otherwise be not relied upon.

3. ### Data Flow {#data-flow}

As previously specified, the entry\_points.txt file identifies the entrypoint to load and call.

4. ### Error Handling {#error-handling}

Exceptions when loading an entrypoint’s on\_startup should be caught and treated as a failure to initialize the validator.

Exceptions when calling on\_shutdown should similarly be reported as a failure, even if the asset otherwise passed tests. A missing on\_shutdown should be treated as a failure.

5. ### Logging and Debugging {#logging-and-debugging}

Omniverse Asset Validator uses the python logging module. A message with level INFO should alert the user that an entrypoint module is being loaded, started, and shutdown.

```
INFO:Loading omni.asset-validator entrypoint: 'some_rules:Registrant'INFO:Calling 'some_rules.Registrant.on_startup()'INFO:Calling 'some_rules.Registrant.on_shutdown()'
```

For debugging, an isolation list may be specified as an environment variable to suppress the loading of an entrypoint. Asset validator entry points should be considered an implementation detail. They are intentionally not expressed through the command line interface or configuration files.

```shell
OMNI_ASSET_VALIDATOR_ISOLATE_ENTRYPOINTS=my_rules:Registrant
```

Users should be warned when an entrypoint is skipped.

```
WARNING:Skipping omni.asset-validator entrypoint module: 'my_rules'
```

6. ### State Machine {#state-machine}

Not applicable

4. ## Security Design {#security-design}

   1. ### Lockfile Pinning

Users and integrators should use a lockfile (‘uv.lock’, ‘poetry.lock’, or equivalent) when installing entrypoint packages, as this cryptographically binds installed packages to verified versions, preventing silent substitutions by compromised or typosquatted packages.

2. ### Package Provenance

Entrypoint packages should be installed only from known, trusted publishers.

5. ## Observability Design

Not applicable

6. ## High Availability

Availability of plugins delegated to the package manager indices users are using.

7. ## Scalability

There should be little to no overhead to loading an plugin module indirectly through importlib instead of explicitly importing them.

8. ## Test Automation {#test-automation}

The following test cases should be exercised on CI.  The test plugin modules should be locally defined and not depend on an external package index.

1. ### Verify Self Contained Plugin Package

Verify that the expected set of requirements, features, validators, and profiles are present when registering a self contained plugin module.

2. ### Verify Module With External Dependencies

Verify that the expected set of requirements, features, validators, and profiles are present when registering a module with dependent modules (specified via requirements.txt).

3. ### Verify Invalid Plugin Are Handled Gracefully

Verify that the asset validator command line exits gracefully when registering a module that generates an error on load via importlib.

Verify that the Kit asset validator user interface logs an error message on startup that a plugin module failed to load.

9. ## Other Design Considerations {#other-design-considerations}

   1. ### Resource Limits {#resource-limits}

N/A

2. ### Future Work {#future-work}

Future work should address broader OpenUSD C++ plugin availability, including validator plugins.

Future work may entail making profiles importable for other frameworks like transformers.

A future revision of the asset validator should determine if code generated python is necessary to register requirements, features, and profiles in lieu of using importlib.resources to pull in declarative JSON or YAML.  This would warrant a major version increase.

A future revision of the entrypoint interface may decide to break from the Kit paradigm of on\_startup/on\_shutdown in favor of returning the checkers, profiles, etc. directly. This reduces the burden on the user from having to reason about module load order and whether or not rules are loaded in on\_startup or via imports.