---

# **Gradio-Based Sensor Data Visualization - Failed Attempt**

`src/failed_attempts/gradio_graph_attempt.md` documents a failed attempt to enhance the existing **sensor data retrieval functions** by adding **graph generation capabilities**. 

For details on functions like `moisture_sensor_info`, `light_sensor_info`, and `dht_sensor_info`, refer to the **chatbot_ui.md** documentation.

This document outlines the **attempt to generate and display graphs using Gradio UI**, the **issues encountered**, and serves as a reference for debugging and improving future implementations.

---

## **Overview of the Attempt**

In this attempt, the `graph_from_file` function was added to the existing Gradio chatbot to **generate graphs from CSV data** and display them in the chatbot UI.

- **New Feature**: The `graph_from_file` function samples sensor data and creates a graph.  
- **Problem**: The graph image was not displayed in the Gradio UI, despite being saved successfully.

---

## **1. Installation and Execution**

**Required Libraries Installation**  
```bash
pip install gradio pandas matplotlib openai
```

**Run the Script**  
```bash
python script_name.py
```

---

## **2. Newly Added Function**

### **graph_from_file**

This function reads data from a CSV file, **samples it, and generates a graph**.

**Code**:
```python
import pandas as pd
import matplotlib.pyplot as plt

def graph_from_file(file_path, x_column, y_columns, sample_step=27):
    try:
        # Read CSV file
        data = pd.read_csv(file_path)
        
        # Sample data: Skip rows by sample_step
        sampled_data = data.iloc[::sample_step, :].reset_index(drop=True)

        # Remove % and C symbols from numeric data
        for column in y_columns:
            if '%' in sampled_data[column].iloc[0]:
                sampled_data[column] = sampled_data[column].str.rstrip('%').astype(int)
            elif 'C' in sampled_data[column].iloc[0]:
                sampled_data[column] = sampled_data[column].str.rstrip('C').astype(int)

        # Generate graph
        plt.figure(figsize=(10, 6))
        for column in y_columns:
            plt.plot(sampled_data[x_column], sampled_data[column], label=column)
        
        plt.xlabel(x_column)
        plt.ylabel("Values")
        plt.title("Sampled Sensor Data Visualization")
        plt.xticks(rotation=45)
        plt.legend()
        plt.tight_layout()

        # Save graph
        graph_path = "sensor_data_graph.png"
        plt.savefig(graph_path)
        plt.close()

        return {"graph_path": graph_path, "message": "Graph created successfully with sampled data."}
    except Exception as e:
        return {"error": str(e)}
```

---

## **3. Updated `sensor_functions`**

The `sensor_functions` array now includes the **`graph_from_file`** function.

```python
sensor_functions = [
    # Refer to chatbot_ui.md for existing functions like moisture_sensor_info, etc.
    {
        "type": "function",
        "function": {
            "name": "graph_from_file",
            "description": "Generates a graph using data from a CSV file with specified x and y columns.",
            "parameters": {
                "type": "object",
                "properties": {
                    "file_path": {
                        "type": "string",
                        "description": "The path to the CSV file containing the sensor data."
                    },
                    "x_column": {
                        "type": "string",
                        "description": "The column name to use as the x-axis (e.g., 'Timestamp')."
                    },
                    "y_columns": {
                        "type": "array",
                        "items": {"type": "string"},
                        "description": "A list of column names to use as the y-axis (e.g., 'Soil Moisture (%)')."
                    },
                    "sample_step": {
                        "type": "integer",
                        "description": "The step size for sampling data points to reduce graph density. Default is 27.",
                        "default": 27
                    }
                },
                "required": ["file_path", "x_column", "y_columns"]
            }
        }
    }
]
```

---

## **4. Gradio UI**

Gradio UI integrates the `graph_from_file` function, allowing users to input sensor data queries and display results.

```python
import gradio as gr
import json
from openai import OpenAI

# Function to process user queries
def ask_openai(llm_model, messages, user_message, functions):
    client = OpenAI()
    proc_messages = messages.copy()

    if user_message:
        proc_messages.append({"role": "user", "content": user_message})

    response = client.chat.completions.create(
        model=llm_model, messages=proc_messages, tools=functions, tool_choice="auto"
    )
    response_message = response.choices[0].message
    tool_calls = response_message.tool_calls

    if tool_calls:
        available_functions = {
            "graph_from_file": graph_from_file  # Newly added function
        }

        proc_messages.append(response_message)

        for tool_call in tool_calls:
            function_name = tool_call.function.name
            function_to_call = available_functions.get(function_name)
            if function_to_call:
                try:
                    function_args = json.loads(tool_call.function.arguments)
                    function_response = function_to_call(**function_args)
                    proc_messages.append({
                        "tool_call_id": tool_call.id,
                        "role": "tool",
                        "name": function_name,
                        "content": json.dumps(function_response),
                    })
                except Exception as e:
                    proc_messages.append({
                        "tool_call_id": tool_call.id,
                        "role": "tool",
                        "name": function_name,
                        "content": json.dumps({"error": str(e)}),
                    })

        second_response = client.chat.completions.create(
            model=llm_model, messages=proc_messages
        )
        assistant_message = second_response.choices[0].message.content
    else:
        assistant_message = response_message.content

    proc_messages.append({"role": "assistant", "content": assistant_message})
    return proc_messages, assistant_message

# Gradio interface
messages = []

def process(user_message, chat_history):
    proc_messages, ai_message = ask_openai("gpt-4o", messages, user_message, functions=sensor_functions)
    chat_history.append((user_message, ai_message))
    return "", chat_history

with gr.Blocks() as demo:
    gr.Markdown("# Sensor Data Retrieval and Graph Generation")
    chatbot = gr.Chatbot(label="Sensor Data and Visualization")
    user_textbox = gr.Textbox(label="Input")
    user_textbox.submit(process, [user_textbox, chatbot], [user_textbox, chatbot])

demo.launch(share=True, debug=True)
```

---

## **5. Example Execution**

### **Input Example**
```
"Generate a graph from a CSV file. Use Timestamp for the x-axis and Soil Moisture (%) and Temperature (C) for the y-axis."
```

### **Expected Output**
- Graph image: `sensor_data_graph.png`  
- X-axis: `Timestamp`  
- Y-axes: `Soil Moisture (%)`, `Temperature (C)`

---

## **6. Analysis of Failure**

### **Issue**
- While the Gradio server executed successfully and the graph image was saved, the graph did not appear in the chatbot UI.  
- The server logs confirmed the graph was generated and saved, but only text responses were shown in the UI.

### **Potential Causes**
1. **Gradio Chatbot Limitation**:  
   - The `Chatbot` component in Gradio is optimized for **text responses**.  
   - Displaying images requires using a separate `gr.Image` component.

2. **File Path Issues**:  
   - The generated graph `sensor_data_graph.png` was saved using a relative path, which might not have been accessible to the Gradio UI.

3. **FRPC File Missing**:  
   - Sharing links failed because the required **frpc_linux_aarch64_v0.2** file was missing.  
   - This file is essential for external access to the local Gradio server.

---

## **7. Attempted Fixes**

- **Attempt 1**: Use `gr.Image` to display the generated graph.  
- **Attempt 2**: Save the graph file using an absolute path.  
- **Attempt 3**: Manually download the missing FRPC file and place it in the specified path.  

Despite these fixes, the **Gradio chatbot still failed to display the image**.

---

## **8. Purpose of Documenting Failures**

1. **Analyze Root Causes**:  
   Recording the reasons for failure helps avoid repeating the same mistakes.

2. **Future Improvements**:  
   - Research better methods for displaying images in Gradio using `gr.Image` or other components.  
   - Ensure the FRPC file is correctly installed to enable sharing functionality.  

3. **Learning Resource**:  
   Failed attempts provide valuable insights into **debugging** and **problem-solving strategies**.

---

## **9. Next Steps**

1. Consult Gradio documentation to identify suitable methods for **image display**.  
2. Install the missing FRPC file and verify sharing functionality.  
3. Check for file access or path issues and make necessary adjustments to the code.

---
