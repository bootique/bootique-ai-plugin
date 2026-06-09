<!--
	Licensed to ObjectStyle LLC under one
	or more contributor license agreements.  See the NOTICE file
	distributed with this work for additional information
	regarding copyright ownership.  The ObjectStyle LLC licenses
	this file to you under the Apache License, Version 2.0 (the
	"License"); you may not use this file except in compliance
	with the License.  You may obtain a copy of the License at

	  http://www.apache.org/licenses/LICENSE-2.0

	Unless required by applicable law or agreed to in writing,
	software distributed under the License is distributed on an
	"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
	KIND, either express or implied.  See the License for the
	specific language governing permissions and limitations
	under the License.
-->
# bootique — Claude Code plugin

Bootique workflows for Claude Code.

This plugin teaches Claude how to build and extend [Bootique](https://bootique.io)
applications — bootstrapping a runnable app, authoring modules, wiring services
through DI, defining configuration, and writing commands.

The plugin targets downstream **users of Bootique** building their own Java apps,
as well as authors of reusable Bootique modules.

## Install

The plugin is distributed from the Bootique GitHub repository:
**https://github.com/bootique/bootique-ai-plugin**. Inside Claude Code:

```
# register the "marketplace"
/plugin marketplace add bootique/bootique-ai-plugin

# install the plugin
/plugin install bootique@bootique
```

## What's in here

```
plugin/
├── .claude-plugin/plugin.json   # manifest
├── README.md                    # this file
├── skills/                      # auto-triggering workflows
└── references/                  # source-of-truth docs loaded by skills
```

Each skill is a thin trigger that loads the right reference docs and walks Claude
through the workflow.
