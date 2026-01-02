
Q. Your RAG system is hallucinating in production. How do you diagnose what's broken - the retriever or the generator?
Ans: The fundamental insight: 
```
RAG quality = Retriever Performance × Generator Performance
```

If either component scores zero, your entire system fails. It's multiplication, not addition. You can't compensate for bad retrieval with a better LLM.

Retrieval Metrics (Did we get the right context?)  
- Contextual Relevancy: What % of retrieved chunks actually matter?  
- Contextual Recall: Did we retrieve ALL the info needed?  
- Contextual Precision: Are relevant chunks ranked higher than junk?

Generation Metrics (Did the LLM use context correctly?)  
- Faithfulness: Is the output contradicting the retrieved facts?  
- Answer Relevancy: Is the response actually answering the question?  
- Custom metrics: Does it follow your specific format/style requirements?

Here's the diagnostic framework every senior ML engineer knows: 
```
High faithfulness + Low relevancy = Retrieval problem 
Low faithfulness + High relevancy = Generation problem 
Both low = Your entire pipeline is broken 
Both high = Look for edge cases
```

The metric that catches most production issues: Contextual Recall Your retriever might find "relevant" content but miss critical details. 
Perfect precision, zero recall = confident wrong answers. This is why RAG systems confidently hallucinate. "Our RAG has 85% accuracy!" 

Interviewer: "What's your contextual precision? Faithfulness score? Are you measuring end-to-end or component-level?" Vague metrics = You don't understand production RAG systems.

Know your evaluation targets by use case: Customer support: 
Faithfulness >0.9 (no wrong info) 
Research assistant: Contextual recall >0.8 (comprehensive) 
Code completion: Answer relevancy >0.9 (stay on topic) 
Legal docs: All metrics >0.95 (zero tolerance)

The brutal production reality: Perfect retrieval + weak prompts = hallucinations Perfect LLM + bad chunks = irrelevant answers 
Good retrieval + good generation + no monitoring = eventual failure 
You need metrics at ALL stages.

Mention LLM-as-a-judge evaluation "I'd use GPT-4 to evaluate faithfulness by comparing generated answers against retrieved context, then track score distributions over time to catch model drift."


"How would you implement this evaluation in production?" 
Wrong: "Run tests manually" 
Right: - Automated component-level evals in CI/CD 
- real-time monitoring with alerting 
- async batch evaluation of production traffic"

