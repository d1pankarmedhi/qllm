<div align="center">
<h1>nn-linear-quantization</h1>
<h3>Linear Quantization of neural networks </h3>
</div>

The motivation behind this tool is to reduce model size and memory usage for inference. Storing large models can be burdensome, especially when dealing with edge devices. A model with a smaller footprint often simplifies and streamlines the inference process.

It allows developers to quantize any Hugging Face model from the HF Hub without losing much in terms of performance and accuracy. The underlying method is a simple linear quantization technique (both asymmetric and symmetric), enabling quick and ready-to-go quantization on the fly.

> **Note**: This tool is not ready for production environments and should only be used for building intuition. Consider it a tool for model quantization experimentation.

## Getting Started

### Usage

1.  **Download and Load a Pre-trained Model:**
       
    ```Python
    from transformers import AutoModelForCausalLM, AutoTokenizer, pipeline
    import torch
    
    model_id = "facebook/opt-350m"
    model = AutoModelForCausalLM.from_pretrained(
        model_id,
        torch_dtype=torch.bfloat16,
        low_cpu_mem_usage=True
    )
    tokenizer = AutoTokenizer.from_pretrained(model_id)
    
    # Get the original model size   
    print(f"Original footprint of the model: {model.get_memory_footprint()/1e+6} MB")
    ```

    ```
    # The Original footprint of the model: 662.392832 MB
    ```




2.  **Run and Test the Original Model:**
     
    ```Python
    pipe = pipeline("text-generation", model=model, tokenizer=tokenizer)
    print(pipe("What are we having for dinner?"))
    
    ```
    
    ```
    # [{'generated_text': "What are we having for dinner?\nI'm having a steak and a salad.\nI'm"}]
    
    ```
    
3.  **Load and Quantize Linear Layers:**
    
    ```Python
    from qllm import LinearQuantizer
    from qllm.layers import W8A16LL
    
    LinearQuantizer.replace_and_quantize_modules(model, W8A16LL, ['lm_head'])
    
    # Get the quantized model size
    print(f"Model footprint after quantization: {model.get_memory_footprint()/1e+6} MB")
    ```
    
    ```
    # Model footprint after quantization: 359.799808 MB
    ```
    
4.  **Test the Quantized Model:**
        
    ```Python
    pipe = pipeline("text-generation", model=model, tokenizer=tokenizer)
    print(pipe("What are we having for dinner?"))
    
    ```
    
    ```
    # [{'generated_text': "What are we having for dinner?\nI'm having a steak dinner.\nI'm having a"}]
    
    ```
    

## Observation

There is around a `45.68%` reduction in model footprint. This may not be the same for every model, but a reduction of anywhere around 30% is significantly helpful from the inference point of view.

## Key Features

-   **Lightweight Quantization:** Utilizes simple linear quantization techniques for efficient model compression.
-   **Flexibility:** Supports both asymmetric and symmetric quantization.
-   **Layer-Specific Quantization:** Allows targeting specific linear layers for quantization (e.g., `lm_head`).

### Linear Quantization

-   **Symmetric Quantization:** The range of quantized values is symmetric around zero. This is simpler but might be less accurate if the data is not centered around zero.
-   **Asymmetric Quantization:** The range of quantized values is not necessarily symmetric around zero and uses a zero-point offset. This can provide better accuracy for data that is not centered around zero.

The `W8A16LL` layer indicates a quantization scheme where weights are quantized to 8 bits (W8) and activations remain in 16-bit floating-point (A16) during the forward pass within the quantized linear layer.

## Contribution 

We warmly welcome contributions to the `qllm` project! Whether you're interested in discussing quantization techniques or talk about neural nets in general, you can start a discussion. 
