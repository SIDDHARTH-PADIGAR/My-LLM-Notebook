# **Building My Own LLM from Scratch – My Journey & Learnings**  

## **Introduction**  
This project was an **ambitious deep dive** into **building, training, and fine-tuning my own LLM from scratch.** Unlike using pre-trained models, I wanted to **understand the entire process**—from architecture to training pipelines and inference.  

This documentation reflects **my personal experience, learnings, and technical decisions** throughout the process.  

---

## **1. My Transformer Architecture**  
Before diving into training, I had to **understand how transformer-based LLMs work** at a fundamental level.  

### **The Model**  
- I built my model **based on the Transformer architecture** with **multi-head self-attention**.  
- The **core** of my model consisted of **12 Transformer blocks**, just like in the GPT-2 architecture.  
- Each block had:  
  - **Multi-head attention layers** to capture word relationships.  
  - **Feed-forward networks** to process and refine attention outputs.  
  - **Layer normalization & residual connections** to stabilize training.

  ### **Multi-Head Attention – The Core of My Transformer**  

One of the most crucial components of my LLM was **multi-head attention (MHA)**. It’s what allows the model to **understand context, relationships, and dependencies between words**—even if they’re far apart in a sentence.  

---

### **How It Works**  
Instead of relying on a **single attention mechanism**, MHA **splits the input into multiple attention heads**. Each head learns **different aspects of the input** and collectively provides a richer representation.  

#### **Step-by-Step Breakdown**  
1. **Input Embeddings → Query, Key, Value (Q, K, V) Matrices**  
   - Each token in the input sequence is projected into three separate matrices:  
     - **Query (Q):** What this word is looking for.  
     - **Key (K):** How relevant other words are to this word.  
     - **Value (V):** The actual information passed forward.  
   
2. **Scaled Dot-Product Attention**  
   - Attention scores are computed using:  
     \[
     \text{Attention}(Q, K, V) = \text{softmax} \left( \frac{QK^T}{\sqrt{d_k}} \right) V
     \]
   - This determines **which words influence each other** in the sequence.  

3. **Multiple Attention Heads Compute in Parallel**  
   - Instead of a single set of (Q, K, V), I used **multiple heads** (e.g., 12 heads in my model).  
   - Each head captures different aspects of meaning, like:  
     - **One head might focus on syntax (e.g., subject-verb agreement).**  
     - **Another might focus on long-range dependencies (e.g., linking words in long sentences).**  

4. **Concatenation & Final Projection**  
   - All attention heads are **concatenated and projected** back into the model’s embedding space.  
   - This allows the model to **learn richer context-aware representations** instead of a single, narrow attention focus.  

---

### **Why Multi-Head Attention Was Crucial for My Model**  
✔ **Prevents Overfitting to One Type of Relationship** – Instead of one attention map, I got multiple diverse perspectives.  
✔ **Improves Context Awareness** – The model **understands both local (nearby words) and global (long-range dependencies) context.**  
✔ **Boosts Expressiveness** – Since each head specializes in different relationships, my model produced more nuanced responses.  

> *Think of it like having multiple experts analyze a sentence from different perspectives—grammar, semantics, intent, and more. Their combined knowledge gives a deeper understanding.*  

This was one of the **key reasons my transformer architecture worked so well** in learning **meaningful responses** during training. 🚀

### **The Training Strategy**  
- **Tokenization:** Used **Byte Pair Encoding (BPE)** to efficiently handle vocabulary.  
- **Batch Processing:** Instead of training on individual examples, I **batched** them to speed up learning.  
- **Checkpointing:** I saved my model at regular intervals to avoid losing progress.  

> *One of the key realizations here was that training an LLM isn’t just about feeding data—it’s about structuring the architecture so it learns meaningfully.*  

---

## **2. Fine-tuning – Two Different Approaches**  
Once the base transformer model was trained, I had to **fine-tune it for specific tasks**. There were **two main approaches** I experimented with:  

### **A. Classification Fine-Tuning**  
- Used for tasks like **spam detection**, where the model had to **classify** inputs into distinct categories.  
- **Focused only on the last output token** rather than the full sequence.  
- **Loss Function:** Standard **Cross-Entropy Loss**, since it’s great for categorical predictions.  
- **Key Learning:** **This approach was rigid**—the model only learned to differentiate classes but wasn’t able to generate complex outputs.  

### **B. Instruction Fine-Tuning**  
- This was the **game-changer.**  
- Instead of classification, I **trained the model to follow human-like instructions**.  
- The **entire sequence mattered**, not just the last token.  
- **Loss Function:** Still **Cross-Entropy Loss**, but applied across the sequence, allowing the model to learn from example responses.  
- **Key Learning:** This approach was **way more flexible** and helped the model generate **coherent, structured responses** instead of just labels.  

---

## **3. Masking vs. Not Masking Instructions – My Take**  
One of the biggest conceptual debates in instruction fine-tuning was:  

📌 **Should I mask instructions (ignore them in training loss), or should I let the model learn from them?**  

### **Option 1: Masking Instructions**  
- Here, I **excluded the instruction text** from the loss calculation.  
- The model **only learned from the responses**, ensuring it **focused purely on generating correct answers**.  
- **Downside:** The responses **sometimes felt disconnected**—like the model was answering without fully understanding the context.  

### **Option 2: Not Masking Instructions (What I Chose)**  
- I **included instructions in training**, allowing the model to learn **both the question and the response together.**  
- **Why?**  
  - A person who remembers both **questions and answers** tends to **give better, more contextual responses**.  
  - If a model understands **how a question is phrased**, it can **generate better answers**.  
- **Key Learning:** Not masking instructions helped my model become more **adaptive and natural in its responses.**  

> *I realized that overly rigid answers can make a model feel robotic, just like how someone who only answers “yes” or “no” feels cold in a conversation.*  

---

## **4. Training Process & Hyperparameters**  
I went through **multiple iterations** of training, tweaking hyperparameters to get the best possible results.  

### **Key Hyperparameters & Their Role**  
- **Learning Rate (lr = 0.00005):** Controlled how aggressively the model adapted to new data.  
  - **Metaphor:** A student studying too fast (high lr) will **memorize but not understand**, while a student learning too slow (low lr) will **take forever to finish the syllabus**.  
- **Weight Decay (0.1):** Prevented overfitting by discouraging memorization.  
  - **Metaphor:** A professor penalizing students for just **memorizing content instead of actually learning.**  
- **Batch Size:** **Controlled memory efficiency**—too big, and I’d get memory crashes; too small, and training would be slow.  

---

## **5. The Ollama Deployment Struggle**  
Once my model was trained, I needed a way to **run inference** locally. **Enter Ollama.**  

### **The Issues I Faced**  
1. **Port Binding Errors** (`Only one usage of each socket address is permitted`).  
2. **Memory Crashes** (`Model requires 8.4 GiB but only 4.2 GiB available`).  
3. **Redundant Retraining** every time Jupyter restarted.  

### **Final Result?**  
- **Couldn’t test LLaMA 3 due to memory limits.**  
- **Switched to LLaMA 2 7B, but even that was too heavy.**  
- **Confirmed that my model was correctly loaded but couldn’t run full inference locally.**  

📌 **Lesson Learned:** Model deployment is as much about **hardware limitations** as it is about model performance. **I need cloud GPUs for real testing.**  

---

## **6. What I Learned (Beyond Just AI)**  
This project wasn’t just about **building a model**—it taught me **how to solve problems systematically**.  

### **Technical Takeaways:**  
✔️ **Transformer architectures aren’t magic**—they are carefully stacked attention layers.  
✔️ **Batching data is crucial**—it prevents memory overload.  
✔️ **Hyperparameters control everything**—finding the right balance is key.  
✔️ **Ollama & API-based inference is powerful** but requires proper setup.  

--- 

### **Immediate Next Steps:**  
- **Test inference on a cloud GPU** (AWS / Google Colab).  
- **Try quantization techniques** to reduce memory usage.  
- **Optimize my training pipeline** to handle larger datasets efficiently.  

### **Final Thought:**  
> *This wasn’t just about training a model. It was about proving to myself that I could build something from scratch, understand it deeply, and optimize it for real-world use. The journey isn’t over—this is just the beginning.*  
