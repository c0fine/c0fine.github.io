This blog will walk you though setting up all of the required dependencies to build your own MCP server to work with open source tools. 

## Installing dependencies 

### Homebrew
1. If you don't already have it install homebrew 
`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"` . If you can't run the command visit https://github.com/Homebrew/brew/releases to use the installer instead.

### Ollama
1. Install `Ollama` 
	either via downloading the app at https://ollama.com/download or `brew install ollama`
2. Pull down the model you want to use. This blog will use `llama3.1:8b` as most M-Series should be able to handle it but the largest **instruct** model you can comfortably run is probably best. I'll put out a new blog soon with performance comparisons with various Ollama models
	1. `ollama pull llama3.1`
3. Start the ollama server 
	1. `ollama serve`
4. Validate ollama server is running
	1. `localhost:11434` which should show `Ollama is running` in the browser

### MCP and Other Dependencies 
1. Install `uv`
	1. `curl -LsSf https://astral.sh/uv/install.sh | sh` if that doesn't work visit https://docs.astral.sh/uv/getting-started/installation/#standalone-installer for alternatives
2. Node.js is required for various development tools 
	1. `brew install node@22` If that doesn't work visit https://nodejs.org/en/download for alternatives

### Installing and Setting Up Goose
1. Install the GUI app for MacOS
	1. Navigate to https://block.github.io/goose/docs/getting-started/installation and select download under Goose Desktop. if you're not on Mac you'll have to use the CLI version. the server we're going to write will work but I'm not going to over how to add it to CLI Goose in this post. 
2. Add Ollama to Goose
	1. Launch the Goose app and select the grey plus button in the lower right corner of the Ollama box. If you set up the Ollama server correctly earlier you'll get a green checkbox next to where it says Ollama. If it doesn't work try downloading the Ollama app from here https://ollama.com and launch it then try again. 
3. Configure Goose to use `llama3.1:8b`
	1. In the upper right corner of the chat window enter the settings window by click the 3 dots, or hit command + comma
	2. Click browse to the right of the `Models` header. Right now it's probably only showing `quen2.5`
	3. Under `Add Model` click the `Select provider` dropdown and click `Ollama`
	4. Under `Model name` enter `llama3.1:8b`

### Setting up the Repo
Make sure whenever we're naming things (indicated by side carrots) you use a meaningful name. The agent will be able to read all of the names and it will contribute to the context that the model uses to make decisions. Especially with open-source clients and models it will make it less useable with bad nomenclature 
1. Initialize the repo
	1. run `uv init <reponame>` 
2. Move into the directory
	1. `cd <reponame>`
3. create virtual environment 
	1. `uv venv`
4. Activate virtual environment
	1. `source .venv/bin/activate`
5. Add dependencies, for this post we're not using anything extra but this is where you'd add other packages you need
	1. `uv add "mcp[cli]" <othermodules>`
6. Remove the auto-generated . py files
	1. `rm *.py`
7. Create your server file 
	1. `touch server.py`

## Add template code
There is some parts that every MCP server write should have. These things make it easier for the model to understand your code and improve its performance. 
Make sure when you write your tools (the functions annotated with `@mcp.tool()`) must be explicitly typed, must be meaningfully named, and must have really detailed docstrings (the comments in the triple quotes). The LLM consumes this information as added context to create better responses.
```
from mcp.server.fastmcp import FastMCP  
from mcp.shared.exceptions import McpError  
from mcp.types import ErrorData, INTERNAL_ERROR, INVALID_PARAMS

mcp = FastMCP("meaningful_name") #make sure this is a meaningfull name 
@mcp.tool() #you have to have this annotation for your tools to be useable
def clear_descriptive_function_name(arguement: type) -> returntype:
	"""
	Full description of what the funciton does in big blocks. Eg: Fetches data about a given pokemon. 
	Parses the returned data for all known moves, creates a list of all moves, returns list of moves. 
	Useage:
		clear_descriptive_function_name(arguement)
		eg: get_pokemon_moves("Turtwig")
	"""
	try: 
		#do input validation and provide meaningfull error messages and raise them with MCP error messages. this will also provide the model additional context to better answer your prompts. 
		return ret_value
except ValueError as e:  		
	raise McpError(ErrorData(INVALID_PARAMS, str(e))) from e  
except RequestException as e:  
	raise McpError(ErrorData(INTERNAL_ERROR, f"Request error: {str(e)}")) from e  
except Exception as e:  
	raise McpError(ErrorData(INTERNAL_ERROR, f"Unexpected error: {str(e)}")) from e
```

## Testing your Code
MCP provides a development environment to ensure your tool responds as expected without dealing with the added variance from the AI model. 
1. In the directory with `server.py` run 
	1. `npx @modelcontextprotocol/inspector uv run server.py`
2. If you don't already have inspector hit y to install it
3. It will then start inspector, click the provided url 
4. On the left side hit connect
5. In the top bar navigate to `Tools`
6. Click `List Tools`
7. It should then list the tools 
8. For each tool provide it arguments and press run to make sure it works

## Add your server to Goose
1. Navigate back to the settings page
2. Under extensions click `Add custom extension`
3. Under `Type` select `StandardIO`
4. Enter the name you gave the MCP server in the `mcp = FastMCP()` line in under `ID` and `Name`
5. Enter a short meaningful description in `Description`
6. Under command enter
	1. `uv run /absolute/path/to/server.py`

Now you can use your selected model to call all of the tools you've made. 
