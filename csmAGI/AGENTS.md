## Project Overview

## Implementation Rules
- The agent's working directory scope is limited to `/csmAGI`
- `/csmAGI/Reference` contains a basic introduction to CSM, descriptions of relevant commands, and format definitions. Don't modify the exist files under this path.
- Each second-level subdirectory within `/csmAG/csm` represents a fully developed CSM module, with relevant documentation provided in the included Markdown files. 
- All generated scripts are saved in `/csmAGI/scripts`. Named in format `[timestamp]-[general function].csmscript`
- When an agent is required to design a new module, just need to create the module directory under `/csmAG/csm` with similar naming rulers,and generate the content of the module's Markdown file. No need to create LabVIEW code.