# Create your own agent workflow

## Introduction

If you want to learn how to create an Agent workflow we have provided all the steps in this lab to achieve that.

Estimated Time: 2 hours

### Objectives

This lab will take you through the steps needed to create your own agent workflow

### Prerequisites

This lab assumes you have:

* An Oracle Cloud account
* Administrator permissions or permissions to use the OCI tenancy
* Ability to spin-up A10 instances in OCI
* Ability to create resources with Public IP addresses (Load Balancer, Instances, OKE API Endpoint)
* Access to HuggingFace, accept selected HuggingFace model license agreement.
* Accept CodeLlama7B HuggingFace model license agreement.
* Access to NGC Catalog for Llama3 and CuOpt NIMs.

## Task 1: Create your workflow

1. In the jupiter environment create a new notebook myagentworkflow.ipynb. Start by importing the required modules and install them(this can differ based on your use case): To install python modules directly from jupiter notebook you can create another cell that it will be run before the cell with the code:

     ```text
       <copy>
       !python3 -m pip install openai
       !python3 -m pip install gradio
       !python3 -m pip install --upgrade jupyter ipywidgets
       !python3 -m pip install requests
       </copy>
    ```

    Import the required modules:

    ```text
       <copy>
       import gradio as gr
       from openai import OpenAI
       import sys
       import requests
       import json
       from config import json_format
       </copy>
    ```

2. Add links to the models endpoints. The models are deployed in OKE and we are using the service name(kubectl get service):

    * agent_base_url = [http://llama3/v1](http://llama3/v1)
    * code_base_url = [http://codellama/v1]([http://codellama/v1])
    * optimization_base_url = [http://cuopt-cuopt-deployment-cuopt-service:5000/cuopt/routes](http://cuopt-cuopt-deployment-cuopt-service:5000/cuopt/routes)

3. Next step is setup the parameters for different models.

    ```text
       <copy>
       agent_model_config = {
       "base_url": agent_base_url,
       "api_key": "dummy",
       "temperature": 0,
       "top_p": 1,
       "max_tokens": 1024,
       "stream": True
        }
        code_model_config = {
        "base_url": code_base_url,
        "api_key": "dummy",
        "temperature": 0,
        "top_p": 1,
        "max_tokens": 256,
        "stream": True
        }
       </copy>
    ```

4. In order to not change the model name every time you will add a new model to the workflow we will have a function in python that will discover the model name and cache the result:

    ```text
       <copy>
       # Cache for discovered models
        cached_models = {}

        def discover_model(client):
            if client in cached_models:
                return cached_models[client]
            available_models = client.models.list()
            if len(available_models.data):
                model = available_models.data[0].id
                print(f"Discovered model is: {model}")
                cached_models[client] = model
            else:
                print("No model discovered")
                sys.exit(1)
            return model
       </copy>
    ```

5. After that we will need to define a function for OpenAI client to interact with different models:

    ```text
       <copy>
        def create_completion(client, messages, config):
            model = discover_model(client)
            if model is None:
                return []
            try:
                return client.chat.completions.create(
                    model=model,
                    messages=messages,
                    temperature=config["temperature"],
                    top_p=config["top_p"],
                    max_tokens=config["max_tokens"],
                    stream=config["stream"]
                )
            except Exception as e:
                print("Error: ", e)
                return []
       </copy>
    ```

6. At this point, all the prerequisites function are defined and we can start to build functions to interact with each model:
Define function for the main agent to decide what agent can provide the best answer:

    ```text
       <copy>
        def get_decision_from_agent(query):
            system_messages = {
                "role": "system",
                "content": f"Please classify the following query into one of the three categories: 'code', 'optimization', or 'general'. 'code' for queries asking for code examples, programming syntax, or specific implementation details. 'optimization' for queries related to improving performance, enhancing efficiency, or refining algorithms. 'general' for queries that do not specifically pertain to coding or optimization but instead involve general concepts, explanations, or broad questions. Answer only with 'code', 'optimization', or 'general':"
            }
            user_messages = {"role": "user", "content": query}
            messages = [system_messages, user_messages]
            response = create_completion(client_agent, messages, {**agent_model_config, "max_tokens": 10})
            decision = ""
            for chunk in response:
                if chunk.choices[0].delta.content is not None:
                decision += chunk.choices[0].delta.content
            return decision.strip().lower()
       </copy>
    ```

    This behavior can be achieved using different prompt engineering techniques. In our example we are defining in the prompt how the decision should be take. Also we are defining each category and it s meaning. Please make sure you are giving clear instruction to the model and test multiple prompts as each model respond differently based on the data that was trained.

7. Define functions to interact with different agents:

    *Code Agent*

     ```text
       <copy>
        def get_response_from_code_model(query):
            messages = [{"role": "user", "content": query}]
            response = create_completion(client_code, messages, code_model_config)
            codemodel_response = ""
            for chunk in response:
                if chunk.choices[0].delta.content is not None:
                    codemodel_response += chunk.choices[0].delta.content
            return codemodel_response.strip().lower()
       </copy>
    ```

    *Main Agent*
    Main Agent will provide answers for general questions:

    ```text
       <copy>
        def get_general_response_from_agent(query):
            messages = [{"role": "user", "content": query}]
            response = create_completion(client_agent, messages, agent_model_config)
            agent_response = ""
            for chunk in response:
                if chunk.choices[0].delta.content is not None:
                    agent_response += chunk.choices[0].delta.content
            return agent_response
       </copy>
    ```

    *Optimization Agent*
    To interact with  optimization Agent(CuOpt) we need to translate the question in a format that the model will be able to understand, which in our case will be a JSON with a predefined structure.
    To generate the JSON we will call the Llama3 model to create it for us based on the incoming question. To teach Llama3 model to generate the JSON we will pass the JSON format and we will give some examples of questions and how the JSON will look like.
    The JSON format is in the config.py file - use the same file or create.
    After the JSON is generated we will send it to CuOpt model using requests module.

    ```text
       <copy>
        def prepare_optimization_input(query):
            messages = [{
                "role": "user",
                "content": f'''You need to generate a payload for cuOpt to be able to answer the Question. Bellow is the structure of the json {json_format}, and you need to capture the values from the Question and populate the JSON.
        The question will be like this: Optimize the routes for three delivery trucks. Truck 1 has a capacity of 4 units and starts at location [0, 0]. Truck 2 has a capacity of 6 units and starts at location [0, 0]. Truck 3 has a capacity of 2 units and starts at location [0, 0]. They need to deliver packages to locations [2, 2] with a demand of 1 unit, [4, 4] with a demand of 3 units, and [6, 6] with a demand of 2 units. All the locations have a time window from 0 to 1080, and the service time at each location is 1 units. The cost to travel between each location is provided in the following cost matrix: from [0, 0] to [2, 2] costs 10, from [0, 0] to [4, 4] costs 20, from [0, 0] to [6, 6] costs 30, from [2, 2] to [4, 4] costs 10, from [2, 2] to [6, 6] costs 20, from [4, 4] to [6, 6] costs 10. This is the expected JSON output for the example question: {{"cost_matrix_data":{{"data":{{"0":[[0,10,20,30],[10,0,10,20],[20,10,0,10],[30,20,10,0]]}}}},"task_data":{{"task_locations":[1,2,3],"demand":[[1,3,2]],"task_time_windows":[[0,1080],[0,1080],[0,1080]],"service_times":[1,1,1]}},"fleet_data":{{"vehicle_locations":[[0,0],[0,0],[0,0]],"capacities":[[4,6,2]],"vehicle_time_windows":[[0,1080],[0,1080],[0,1080]]}},"solver_config":{{"time_limit":2}}}}.
       </copy>
    ```

    Another example of question is:Optimize the delivery routes for two delivery trucks. Truck 1 has a capacity of 15 units and starts at location [0, 0]. Truck 2 has a capacity of 3 units and starts at location [0, 0]. They need to deliver packages to locations [2, 2] with a demand of 2 units and [3, 3] with a demand of 15 unit. Both locations have a time window from 0 to 1080, and the service time at each location is 1 unit. The cost to travel between each location is provided in the following cost matrix: from [0, 0] to [2, 2] costs 10, from [0, 0] to [3, 3] costs 15, from [2, 2] to [3, 3] costs 35. This is the expected JSON for the second example question: {{"costmatrixdata":{{"data":{{"0":[[0,10,15],[10,0,35],[15,35,0]]}}}},"taskdata":{{"tasklocations":[1,2],"demand":[[2,15]],"tasktimewindows":[[0,1080],[0,1080]],"servicetimes":[1,1]}},"fleetdata":{{"vehiclelocations":[[0,0],[0,0]],"capacities":[[15,3]],"vehicletimewindows":[[0,1080],[0,1080]]}},"solverconfig":{{"timelimit":2}}}}.
    Respond only with the JSON. Do not include any additional text.
    When generating the JSON don`t change the provided JSON structure. The only exception is the cost_matrix_data where the number of locations define the dimension of the square matrix. It needs to contains one list for each location, and the length of each list is the total number of locations.
    Learn to identify the correct integers in the question provided as example and generate the JSON for the next question.

    ```text
       <copy>
        Question: {query}'''
        }]
        response = create_completion(client_agent, messages, agent_model_config)
        payload_response = ""
        for chunk in response:
            if chunk.choices[0].delta.content is not None:
                payload_response += chunk.choices[0].delta.content
        return payload_response.strip().lower()
       </copy>
    ```

    ```text
       <copy>
        def get_response_from_optimization(query):
            generated_text = prepare_optimization_input(query)
            log = f"JSON generated by Llama3 that will be passed to Optimization Agent(CuOpt):\n{generated_text}\n----------------------\n"
            print(generated_text)

            try:
                cuopt_payload = json.loads(generated_text)
            except json.JSONDecodeError as e:
                print("Failed to parse generated text:", e)
                cuopt_payload = None
            optimized_routes = "Failed to generate optimized routes."
            if cuopt_payload:
            # Send the payload to cuOpt
                headers = {
                    "CLIENT-VERSION": "custom"
                }
                response_cuopt = requests.post(optimization_base_url, json=cuopt_payload,headers=headers)
                if response_cuopt.status_code == 200:
                    optimized_routes = response_cuopt.json()
                else:
                    print("Failed:", response_cuopt.status_code, response_cuopt.text)
            else:
                print("Invalid payload generated by Agent Workflow.")
            return optimized_routes, log
       </copy>
    ```

8. Now it is time to bring all toghether and define the actual workflow. Bellow we are getting the decision from main agent and based on the type of question the model will be selected and the query will be passed to it:

    ```text
       <copy>
        def get_preliminary_response(query):
            decision = get_decision_from_agent(query)
            log = f"Decision by Agent Workflow: '{decision}'\n----------------------\n"
            if "code" in decision:
                log += "Response generated by: Code Agent(CodeLlama)\n----------------------\n"
                response = get_response_from_code_model(query)
            elif "optimization" in decision:
                response, optimization_log = get_response_from_optimization(query)
                log += optimization_log
                log += "Response generated by: Optimization Agent(CuOpt)\n----------------------\n"
            else:
                log += "Response generated by: General Agent(Llama3)\n----------------------\n"
                response = get_general_response_from_agent(query)
            log += f"Preliminary response:\n{response}\n----------------------\n"
            return response, log
       </copy>
    ```

9. After this, the workflow needs to be able to have all the responses in the same place and the main agent to use the information to generate the response to the end user:

    ```text
       <copy>
        def get_final_response_from_agent_workflow(query, preliminary_response):
            system_messages = {"role": "system", "content": f"Add the {preliminary_response} at the begining of the answer. If this information {preliminary_response}  is a JSON explain each route for each truck, if not generate the answer for the question:"}
            user_messages = {"role": "user", "content": query}
            messages = [system_messages, user_messages]
            final_response = create_completion(client_agent, messages, agent_model_config)
            final_response_text = ""
            for chunk in final_response:
                if chunk.choices[0].delta.content is not None:
                    final_response_text += chunk.choices[0].delta.content
            return final_response_text
       </copy>
    ```

10. Next step is to define clients for each model and setup the api keys that are used to secure the access on the model using Gradio interface to pass them.

    ```text
       <copy>
        def set_api_keys(agent_key, code_key):
            global agent_model_config, code_model_config
            agent_model_config["api_key"] = agent_key
            code_model_config["api_key"] = code_key
            return agent_model_config, code_model_config

        def client_init(agent_config, code_config):
            client_agent = OpenAI(
                base_url=agent_config["base_url"],
                api_key=agent_config["api_key"]
            )
            client_code = OpenAI(
                base_url=code_config["base_url"],
                api_key=code_config["api_key"]
            )
            return client_agent, client_code

        def set_keys_and_init_clients(agent_key, code_key):
            agent_config, code_config = set_api_keys(agent_key, code_key)
            global client_agent, client_code
            client_agent, client_code = client_init(agent_config, code_config)
            return "API keys set and clients initialized."

        api_key_interface = gr.Interface(
            fn=set_keys_and_init_clients,
            inputs=[gr.Textbox(type="password", placeholder="Enter agent API key"), gr.Textbox(type="password", placeholder="Enter code API key")],
            outputs="text",
            title="Set API Keys",
            description="Input the API keys for the agent and code models."
        )
       </copy>
    ```

11. Last step is to create the Gradio interface function and initiate Gradio.

    ```text
       <copy>
        def gradio_interface(query):
            preliminary_response, log = get_preliminary_response(query)
            final_response = get_final_response_from_agent_workflow(query, preliminary_response)
            log += f"Final response:\n{final_response}"
            return log

        workflow_interface = gr.Interface(
            fn=gradio_interface,
            inputs="text",
            outputs="text",
            title="Agent Workflow",
            description="Enter your query below:"
        )

        gr.TabbedInterface([api_key_interface, workflow_interface], ["Set API Keys", "Agent Workflow"]).launch(share=True)
       </copy>
    ```

Following these steps, you will be able to setup from zero an Agent Workflow scenario using Llama3 as main agent, CodeLlama as the code agent and CuOpt as the optimization agent.
If you want to use other models the code gives this possibility but you will need to change the prompts context and to add any additional steps for that might be required by different models.

## Acknowledgements

**Authors**

* **Ionut Sturzu**, Principal Cloud Architect, NACIE
* **Abhinav Jain**, Senior Cloud Engineer, NACIE