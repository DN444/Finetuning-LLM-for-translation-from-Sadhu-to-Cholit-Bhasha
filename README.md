# Finetuning-LLM-for-translation-from-Sadhu-to-Cholit-Bhasha


Bengali has historically evolved into two forms: Sadhu Bhasha and Cholit Bhasha. While Cholit Bhasha is widely used in modern communication, a significant corpus of classical poetry and literature remains in Sadhu Bhasha. Manual conversion between these forms requires linguistic expertise and is time-consuming. Recent advances in Large Language Models (LLMs) pro-vide an opportunity to automate this transformation using data-driven ap-proaches. With this in mind our project aims to fine-tune Google’s Gemma-2B LLM on a dataset of sadhu bhasha Bengali poetry using Low-Rank Ad-aptation (LoRA). We obtain a BLEU of 42.95, chrF of 55.97 and METEOR 0.7844. These results reveal to use the performance of the model used and also suggest the degree of improvement needed to make the model achieve satisfactory performance.


## Training Procedure

The training happens via supervised fine-tuning in which the model is trained to generate the target Cholit Bengali sentence based on the instruction "সাধু ভাষা থেকে চলিত ভাষায় রূপান্তর কর। শুধু চলিত বাক্য দাও।" and an input sentence in Sadhu Bengali. The task is framed as a causal language modelling problem, where the model learns to predict the next token in a sequence.
    Each training sample is formatted as a single concatenated sequence consisting of an instruction, an input sentence, and the expected output. This sequence is tokenized and truncated or padded to a fixed length of 128 tokens.
    During training, our goal is to minimize the cross-entropy loss between the predicted tokens and actual tokens. Loss is computed over the full sequence. The model implicitly learns to condition its predictions on both the instruction and the input sentence.
    The batch size is set to 1.
    The model is trained for 25 epochs, so the entire dataset is iterated over 25 times. This increases the risk of overfitting. No early stopping or validation-based checkpoint selection is used, although validation data is passed during training.
    The optimizer used is AdamW, which separates weight decay from gradient updates. Gradients are computed via backpropagation, and parameter updates are applied using adaptive moment estimates.
    After training, only the LoRA parameters are saved. These represent the learned low-rank adaptations to the base model weights and can be reapplied to the original pretrained model for inference.
    During each forward pass:
•	The tokenized input is embedded and passed through the transformer layers.
•	At each layer, the hidden representations are modified by both the frozen pre-trained weights and the learned LoRA updates.
•	The final hidden states are applied using a linear layer, producing logits for each token position.
    During the backward pass:
•	The cross-entropy loss is computed.
•	Gradients are propagated through the network.
•	Only the LoRA parameters receive gradient updates; the base model weights remain unchanged.
Model performance is evaluated on a test set with up to 200 samples used for evaluation. Predictions are generated using the inference pipeline and compared against reference outputs.
The primary evaluation metrics are the BLEU score and chrF, computed using the sacrebleu library. BLEU measures n-gram overlap between predicted and reference sentences, providing a quantitative assessment of translation quality. chrF does the same but at the character-level n-grams. From nltk we also use METEOR score, which is based on using precision and recall from unigrams.
