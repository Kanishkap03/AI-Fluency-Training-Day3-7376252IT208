Day 3 – Analysis

1. Project Overview

The objective of this project is to build a ReAct-style AI agent from scratch using an LLM and custom tools. The agent follows the basic cycle:

**Reason → Act → Observe → Reason → Final Answer**

The project demonstrates how an agent can use tools such as a calculator and a webpage/file reader instead of guessing information.

The project contains two versions:

- `my_agent.py` – ReAct agent without safety guards.
- `my_agent_fixed.py` – ReAct agent with guards to prevent repeated tool calls without progress.
- `my_tools.py` – Contains the tools used by the agent.
- `notice.html` – Contains course fee information.
- `big.html` – Contains a large attendance register.
- `requirements.txt` – Contains the required Python packages.

---

2. Tools Used

Calculator

The calculator evaluates basic arithmetic expressions.

Example:

```text
(12000 + 18000) * 0.9

Output:

27000.0

The calculator also rejects invalid expressions instead of executing arbitrary Python code.

Example:

Calculator error: invalid syntax

This shows that invalid input is handled safely.

Read Webpage

The read_webpage tool is used to read the contents of a webpage or local HTML file.

For example:

notice.html

was successfully read and returned the course fee information.

When a nonexistent file was requested:

no_such_file.html

the tool returned an error instead of producing a guessed answer.

3. Test 1 – Valid File and Calculation
Question
Read notice.html and tell me the total fee for CS101 and AI202 after the merit scholarship.
Result

The agent successfully called:

read_webpage({'url': 'notice.html'})

The file contained:

CS101 = Rs. 12,000
AI202 = Rs. 18,000
Merit scholarship = 10%

The total before scholarship is:

12000 + 18000 = 30000

After applying the 10% scholarship:

30000 × 0.90 = 27000

The agent returned:

The total fee for CS101 and AI202 after applying the 10 % merit-scholarship reduction is Rs. 27,000.
Observation

The agent successfully used the file-reading tool to obtain the required information and produced the correct total.

4. Test 2 – Missing File
Question
Read fees.html and tell me the fee for CS101.
Result

The tool returned:

Read error: 'fees.html' is not a URL and no such file exists.

The agent responded that it could not locate the file and requested the correct file path or file.

Observation

The agent did not invent a fee for CS101. It correctly reported that the requested file was unavailable.

This demonstrates an important agent behavior:

When required information is unavailable, the agent should report the problem instead of guessing.

5. Test 3 – Large File and Repeated Tool Calls
Question
Read big.html and tell me how many students are listed.
Result in my_agent_fixed.py

The agent called:

read_webpage({'url': 'big.html'})

The tool successfully returned content from the attendance register.

However, the agent repeated the same tool call without making progress.

The guarded version eventually stopped and returned:

Stopped: the tool read_webpage was called 3 times with the same arguments and no progress was made.
Observation

The large file test exposed a limitation in the original agent loop. The model repeatedly requested the same tool with the same arguments instead of changing its approach.

The fixed version detects this repeated behavior and stops the loop.

This prevents the agent from continuing indefinitely.

6. Comparison: No Guards vs Guards
Feature	my_agent.py	my_agent_fixed.py
ReAct loop	Yes	Yes
Tool usage	Yes	Yes
Reads valid files	Yes	Yes
Handles missing files	Yes	Yes
Detects repeated tool calls	No	Yes
Prevents unnecessary looping	No	Yes
Safety exit	Basic maximum-step limit	Additional repeated-call guard
Behavior on big.html	Can repeatedly call the same tool	Stops when no progress is detected
7. Important Difference Observed

For the normal notice.html question, both versions produced the same correct result:

Rs. 27,000

The difference becomes visible when the agent encounters a situation where the tool result does not help it make progress.

For big.html, the model repeatedly requested the same file. The guarded version recognized the repeated action and stopped instead of continuing indefinitely.

Therefore, adding guards improves the reliability of the agent loop.

8. Analysis of my_tools.py

The my_tools.py program was tested separately.

The output was:

27000.0
1024
Calculator error: invalid syntax (<unknown>, line 1). Use only numbers and + - * / ( ).
Fee Notice Department of AI and Data Science...
Read error: 'no_such_file.html' is not a URL and no such file exists.

These tests demonstrate that:

The calculator can perform arithmetic.
The tool can process valid input.
Invalid calculator expressions are rejected.
The webpage reader can read the fee notice.
Missing files produce an error instead of a fabricated result.
9. ReAct Agent Flow

The agent follows these steps:

Step 1 – Reason

The LLM receives the user's question and decides whether a tool is required.

Step 2 – Act

The agent calls a suitable tool such as:

read_webpage

or:

calculator
Step 3 – Observe

The result returned by the tool is added to the conversation.

Step 4 – Reason Again

The LLM uses the tool result to determine the next action or formulate the final answer.

Step 5 – Final Answer

When no additional tool is required, the agent returns the final response.

Step 6 – Guard

In my_agent_fixed.py, additional checks prevent the agent from repeatedly making the same tool call without progress.

10. Key Learning Outcomes

From this experiment, I learned:

How to build a basic ReAct agent from scratch.
How an LLM can select and call tools.
How tool results are returned to the model as observations.
How custom tools can provide information that the model does not know directly.
Why an agent should not guess information that should come from a file or tool.
How invalid tool inputs can be handled.
Why repeated tool calls can cause an agent to get stuck.
How guards can detect repeated actions and stop an unproductive loop.
The importance of a maximum-step limit in an agent loop.
The difference between a basic ReAct agent and a guarded ReAct agent.
11. Conclusion

The Day 3 project successfully demonstrates a ReAct agent built using an LLM and custom tools. The agent was able to read the course fee notice and determine that the total fee for CS101 and AI202 after a 10% merit scholarship is Rs. 27,000.

Testing with a missing file showed that the agent reports unavailable information instead of guessing. Testing with the large attendance file demonstrated that an unguarded agent can repeatedly make the same tool call. The guarded version detects the repeated behavior and stops the execution.

Therefore, the experiment demonstrates that tool use alone is not sufficient for a reliable AI agent; the agent loop also needs appropriate guards and stopping conditions.