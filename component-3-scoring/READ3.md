# Scoring / Relevance Scorer
**Owner:** **Mervin Laforteza**

## Description
This component evaluates the relevance and severity of security threats to the organization. It uses AI models to analyze threat alerts, classify them based on relevance to the organization's technology stack, extract key entities, and assign risk scores.

## Status
- [x] Design done
- [x] Sample data ready
- [x] Start coding
- [x] Testing
- [ ] Connected with other components
- [x] Documentation done

## Key Deliverables

### Week 4: Model Comparison
- **Tested Models:** Sentiment analysis, zero-shot classification, NER, and Llama 3.1
- **Recommended Primary Model:** Llama 3.1 (via Groq) for threat relevance evaluation
- **Recommended Secondary Model:** HuggingFace NER for entity extraction
- **Key Finding:** Llama 3.1 showed highest accuracy and best practical performance at scale

### Week 5: AutoML Fine-Tuning  
- **Task:** Phishing vs Legitimate email classification
- **Model Performance:** 80% accuracy, 100% precision, 71.4% recall, 83% F1 Score
- **Dataset:** 25 training images per class, 5 test images per class
- **Models Evaluated:** Generic sentiment analysis vs fine-tuned transformer models

### Week 7: RAG Security Knowledge Assistant
- **Implementation:** In-memory vector store with document retrieval
- **LLM:** Llama 3.3-70b-versatile via Groq
- **Use Case:** Retrieves security knowledge from threat intelligence documents to provide context-aware threat analysis
- **Test Results:** Successfully answered 5/5 questions with good quality responses

## Technical Architecture
- **Threat Scoring Logic:** Llama 3.1 analyzes alerts and compares against organization's technology stack
- **Entity Recognition:** NER model extracts relevant systems, locations, and organizations
- **Knowledge Base:** RAG system retrieves relevant security standards (MITRE ATT&CK, NIST framework)

## Next Steps
- Integration with n8n workflow for automated scoring
- Connection to Airtable for data persistence
- Real-time threat intelligence feed integration
