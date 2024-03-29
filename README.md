## Libraries and Imports

- `streamlit`: Used to create interactive web applications with Python.
- `PyPDF2`: A library for reading and manipulating PDF files.
- `langchain`: A library for natural language processing tasks.
- `os`: Provides functions for interacting with the operating system.
- `google.generativeai`: Google's Generative AI library.
- `dotenv`: Allows loading environment variables from a .env file.

## Functions

### get_pdf_text(pdf_docs)

1. Function Purpose: This function reads text from each page of the provided PDF documents and concatenates them into a single text string.

```
def get_pdf_text(pdf_docs):
    text = ""
    for pdf in pdf_docs:
        pdf_reader = PdfReader(pdf)
        for page in pdf_reader.pages:
            text += page.extract_text()
    return text
```

- text = "": Initializes an empty string to store the concatenated text from all PDF pages.
- for pdf in pdf_docs:: Iterates through each uploaded PDF document.
- pdf_reader = PdfReader(pdf): Creates a PdfReader object to read the contents of the current PDF document.
- for page in pdf_reader.pages:: Iterates through each page of the current PDF document.
- text += page.extract_text(): Extracts text from the current page and appends it to the text string.
- return text: Returns the concatenated text from all PDF pages.

### get_text_chunks(text)

2. Function Purpose: This function splits the input text into smaller chunks to improve processing efficiency.

```
def get_text_chunks(text):
    text_splitter = RecursiveCharacterTextSplitter(
        chunk_size=10000, chunk_overlap=1000)
    chunks = text_splitter.split_text(text)
    return chunks
```

- text_splitter = RecursiveCharacterTextSplitter(chunk_size=10000, chunk_overlap=1000): Initializes a text splitter object with specified parameters for chunk size and overlap. This helps in breaking down large text into manageable chunks.
- chunks = text_splitter.split_text(text): Splits the input text into chunks using the configured text splitter.
- return chunks: Returns the list of text chunks.

### get_vector_store(text_chunks)

3. Function Purpose: This function generates embeddings for text chunks and creates a vector store using FAISS (Facebook AI Similarity Search).

```
def get_vector_store(text_chunks):
    embeddings = GoogleGenerativeAIEmbeddings(model="models/embedding-001")
    vector_store = FAISS.from_texts(text_chunks, embedding=embeddings)
    vector_store.save_local("faiss_index")
```

- embeddings = GoogleGenerativeAIEmbeddings(model="models/embedding-001"): Initializes a GoogleGenerativeAIEmbeddings object with the specified model for generating embeddings.
- vector_store = FAISS.from_texts(text_chunks, embedding=embeddings): Creates a vector store from the text chunks using the embeddings generated by GoogleGenerativeAIEmbeddings.
- vector_store.save_local("faiss_index"): Saves the vector store locally with the name "faiss_index".

### get_conversational_chain()

4. Function Purpose: This function configures a conversational model for question-answering using Google's Generative AI.

```
def get_conversational_chain():
    prompt_template = """
    Answer the question as detailed as possible from the provided context, make sure to provide all the details, if the answer is not in
    provided context just say, "answer is not available in the context", don't provide the wrong answer\n\n
    Context:\n {context}?\n
    Question: \n{question}\n
    Answer:
    """
    model = ChatGoogleGenerativeAI(model="gemini-pro", temperature=0.3)
    prompt = PromptTemplate(template=prompt_template, input_variables=["context", "question"])
    chain = load_qa_chain(model, chain_type="stuff", prompt=prompt)
    return chain
```

- prompt_template: Defines a template for the conversational prompt, including placeholders for context and question.
- model = ChatGoogleGenerativeAI(model="gemini-pro", temperature=0.3): Initializes a ChatGoogleGenerativeAI model with specified parameters including the model type and temperature.
- prompt = PromptTemplate(template=prompt_template, input_variables=["context", "question"]): Initializes a prompt template object with the defined template and input variables.
- chain = load_qa_chain(model, chain_type="stuff", prompt=prompt): Loads a question-answering chain using the configured model and prompt.
- return chain: Returns the configured conversational chain.

### user_input(user_question)

5. Function Purpose: This function processes user input by finding relevant documents, generating responses using the conversational chain, and displaying the response.
   
```
def user_input(user_question):
    embeddings = GoogleGenerativeAIEmbeddings(model="models/embedding-001")
    new_db = FAISS.load_local("faiss_index", embeddings)
    docs = new_db.similarity_search(user_question)
    chain = get_conversational_chain()
    response = chain({"input_documents": docs, "question": user_question}, return_only_outputs=True)
    print(response)
    st.write("Reply: ", response["output_text"])
```

- embeddings = GoogleGenerativeAIEmbeddings(model="models/embedding-001"): Initializes a GoogleGenerativeAIEmbeddings object for generating embeddings.
- new_db = FAISS.load_local("faiss_index", embeddings): Loads the previously saved vector store using FAISS.
- docs = new_db.similarity_search(user_question): Finds relevant documents based on the user's question using similarity search.
- chain = get_conversational_chain(): Configures a conversational chain for question-answering.
- response = chain({"input_documents": docs, "question": user_question}, return_only_outputs=True): Generates a response using the configured conversational chain.
- st.write("Reply: ", response["output_text"]): Displays the response in the Streamlit interface.

### main()

6. Function Purpose: Sets up the Streamlit web application interface and controls the main functionality of the application.

```
def main():
    st.set_page_config("Chat PDF")
    st.header("Chat with your PDF")
    user_question = st.text_input("Ask a Question from the PDF Files")
    if user_question:
        user_input(user_question)
    with st.sidebar:
        st.title("Menu:")
        pdf_docs = st.file_uploader("Upload your PDF Files and Click on the Submit & Process Button", accept_multiple_files=True)
        if st.button("Submit & Process"):
            with st.spinner("Processing..."):
                raw_text = get_pdf_text(pdf_docs)
                text_chunks = get_text_chunks(raw_text)
                get_vector_store(text_chunks)
                st.success("Done")
```
- st.set_page_config("Chat PDF"): Sets the title of the Streamlit page





