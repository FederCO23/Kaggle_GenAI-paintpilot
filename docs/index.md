# **PaintPilot** - Estimating Paint Like a Pro with GenAI
## Blogpost:

---

## Overview

PaintPilot is an AI-powered assistant that simplifies the process of estimating paint needs and preparing for home renovation. By combining image understanding, structured output, and function calling via Google's Gemini 1.5 Pro and LangGraph, the assistant provides actionable insights just from a photo of a wall.

This project was developed as part of the GenAI Intensive Capstone to showcase how a real-world task like painting can be enhanced with modern generative AI tools, turning a smartphone photo into a smart paint checklist.


<p align="center">
  <img src="./pexels-ivan-samkov-5798974_comp1.jpg" width="400"><br>
  <span style="font-size: 12px; color: gray;">
    Image credit: <a href="https://www.pexels.com/@ivan-samkov/">Ivan Samkov</a> via <a href="https://www.pexels.com">Pexels</a> — Free to use
  </span>
</p>

## Use Case & How It Works

### Goal:
Help a non-expert user answer the question:  
**“How much paint and what tools do I need to repaint this wall?”**

### Steps:
1. **User uploads a photo** of an interior wall.
2. **Scene analysis** is done using Gemini to identify objects (like chairs or windows) and infer wall dimensions.
3. A **structured JSON** response is parsed, estimating height, width, and total paintable area.
4. A **paint estimator function** calculates the quantity of paint required using standard assumptions (2 coats, 10m²/L).
5. A **tool estimator function** recommends brushes, rollers, trays, and accessories based on area size.
6. A **summary assistant** uses the LLM to craft a friendly final message, explaining calculations and prep advice.

---

## Code Snippets

### 1. Scene Analysis with Image Understanding + Structured Output

```python
@tool
def scene_analysis(image_path: str) -> dict:
    image = PILImage.open(image_path)
    response = client.models.generate_content(
        model="models/gemini-1.5-pro-latest",
        config=types.GenerateContentConfig(
            temperature=0.1,
            response_mime_type="application/json",
            response_schema=Wall
        ),
        contents=[prompt, image]
    )
    return response.parsed
```

This generates a response like:

```python
{
  "description": "The image shows a living room...",
  "assumptions": ["Standard ceiling height is 8 feet", "The couch is approximately 6 feet long"],
  "estimated_dimensions": {
    "height": 8,
    "width": 15,
    "area": 120
  }
}

``` 

We use `@tool` to register the `scene_analysis` function as a callable tool for the AI agent. This allows Gemini to automatically analyze an uploaded image, extract visual clues and return a structured JSON with estimated wall dimensions.

### 2. Paint Estimation (Function Calling)

```python
@tool
def paint_estimator(input: SceneInfoInput) -> dict:
    dims = input.scene_info.get("estimated_dimensions", {})
    area_ft2 = dims["area"]
    coats = 2
    coverage_m2_l = 10.0
    paint_liters = (area_ft2 * coats) / coverage_m2_l
    paint_gallons = paint_liters / 3.78541
    ...
```
    
The `@tool` decorator registers `paint_estimator` as a tool that calculates the paint needed based on the estimated wall area. It uses standard assumptions like 2 coats and 10 m²/L coverage, returning the result in both liters and gallons.

### 3. Tool Estimation Logic

```python
@tool
def tool_and_material_estimator(estimation: dict) -> dict:
    area = estimation.get("area_m2", 0)
    return {
        "paint_gallons": estimation.get("paint_gallons", 0),
        "rollers": 1 if area < 50 else 2,
        "brushes": 2,
        "painter_tape_rolls": 1 if area < 30 else 2,
        "drop_cloths": 1 if area < 20 else 2,
        "ladder": area > 25
    }
    
```
 


The `tool_and_material_estimator` tool suggests a checklist of materials based on the wall size—like rollers, brushes, tape, and ladders. Helping users prepare everything they need for painting, based on the estimated area.

### 4. Summary Message via LLM Agent


```python
def chatbot_summary(state: PaintPilotState, llm) -> PaintPilotState:
    prompt = f"""
You are PaintPilot, a renovation assistant...

- Wall area: {area} ft² (approx. {height} ft × {width} ft)
- {coats} coats of paint
- Estimated paint: {liters} liters ({gallons} gallons)
- Tool checklist:
{tool_text}

Explain the paint calculation, give prep advice (e.g., filling holes, taping edges), and be beginner-friendly.
"""
    ...
```

The result is a user-facing natural-language summary that’s accurate, clear, and actionable.


---

## Tech Stack
* Google Gemini 1.5 Pro (Image + JSON generation)

* LangGraph (stateful workflow chaining)

* LangChain tools (function calling support)

* PIL (basic image handling)

* Python + Jupyter + Kaggle runtime


## GenAI Capabilities Demonstrated

| Capability                  | How it’s Demonstrated                                                                 |
|----------------------------|----------------------------------------------------------------------------------------|
| Image understanding         | Gemini 1.5 Pro processes uploaded wall images to infer context and estimate dimensions |
| Structured output / JSON mode | Gemini is prompted to return a JSON with description, assumptions, dimensions         |
| Function calling            | LangChain `@tool` decorators simulate tools with strict inputs, invoked by LangGraph  |
| Agents                      | LangGraph flow manages tool-chaining and state — mimicking agent behavior             |
| Controlled generation       | Prompt engineering guides Gemini to stick to a defined schema (strict JSON)           |
| Gen AI Evaluation           | Gemini evaluates the assistant’s output using a structured rubric and rating system   |




## Future Improvements
* Support multiple images for entire rooms

* Customize outputs based on paint type or wall material

* Integrate interactive clarification questions (“Do you want matte or gloss?”)

* Recommend eco-friendly options


## Conclusion

PaintPilot shows how a simple real-world task, estimating paint needs, can be transformed with modern GenAI capabilities. From image analysis to function-based reasoning and user-friendly summarization, this assistant is a fun and practical application of everything we learned in the GenAI Intensive Course!

Thanks for reading. Built as part of the 2025Q1 Kaggle GenAI Capstone Challenge. 

Special thanks to the Kaggle and Google GenAI team for the course and inspiration. Let’s keep building! 🚀


## Contributors

- [Ben Kern](https://www.linkedin.com/in/benkernconsulting/)
- [Federico Bessi](https://www.linkedin.com/in/federico-bessi/) • [GitHub](https://github.com/FederCO23)


## Citation

@misc{gen-ai-intensive-course-capstone-2025q1,
  author       = {Addison Howard and Brenda Flynn and Myles O'Neill and Nate and Polong Lin},
  title        = {Gen AI Intensive Course Capstone 2025Q1},
  year         = {2025},
  howpublished = {\url{[https://kaggle.com/competitions/gen-ai-intensive-course-capstone-2025q1](https://kaggle.com/competitions/gen-ai-intensive-course-capstone-2025q1)}},
  note         = {Kaggle}
}


