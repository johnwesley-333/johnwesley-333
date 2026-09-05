from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
from langgraph.graph import StateGraph, START, END
from typing import TypedDict
from dotenv import load_dotenv
import os

load_dotenv()

app = FastAPI(title="JOHN AI", version="1.0.0")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


class ChatRequest(BaseModel):
    message: str


class ChatState(TypedDict):
    message: str
    response: str


def ai_agent(state: ChatState):
    message = state["message"].lower()

    if "who is john" in message:
        response = (
            "John Wesley is an AI Developer and B.Tech Artificial Intelligence "
            "and Data Science student at Panimalar Engineering College."
        )

    elif "project" in message:
        response = (
            "John has built AI projects including tiny-langgraph-ai, "
            "an AI Vision System, AI Face Emotion Recognition, "
            "and an AI Object Detection System."
        )

    elif "skill" in message or "technology" in message:
        response = (
            "John works with Python, Java, C, SQL, OpenCV, Scikit-Learn, "
            "LangGraph, FastAPI, DeepFace, Git, GitHub, VS Code and Jupyter."
        )

    elif "learning" in message:
        response = (
            "John is currently learning LangGraph, FastAPI, AI Agents, "
            "RAG, LLM Engineering and System Design for AI Applications."
        )

    elif "career" in message or "goal" in message:
        response = (
            "John's career goals include becoming an AI Engineer, "
            "Machine Learning Engineer, Computer Vision Engineer, "
            "Open Source Contributor and Research-Oriented Developer."
        )

    else:
        response = (
            "I'm JOHN AI. I can answer questions about John's projects, "
            "skills, education, certifications and AI journey."
        )

    return {
        "message": state["message"],
        "response": response
    }


graph_builder = StateGraph(ChatState)

graph_builder.add_node("ai_agent", ai_agent)

graph_builder.add_edge(START, "ai_agent")
graph_builder.add_edge("ai_agent", END)

graph = graph_builder.compile()


@app.get("/")
def root():
    return {
        "name": "JOHN AI",
        "status": "online",
        "message": "JOHN AI is ready."
    }


@app.get("/health")
def health():
    return {
        "status": "healthy",
        "ai": "online",
        "langgraph": "online"
    }


@app.post("/chat")
def chat(request: ChatRequest):
    result = graph.invoke({
        "message": request.message,
        "response": ""
    })

    return {
        "response": result["response"]
    }
import os
from typing import TypedDict

from dotenv import load_dotenv
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel

from langgraph.graph import StateGraph, START, END

from langchain_openai import ChatOpenAI
from langchain_huggingface import HuggingFaceEmbeddings
from langchain_chroma import Chroma
from langchain_core.documents import Document

load_dotenv()

app = FastAPI(
    title="JOHN AI",
    description="RAG-powered personal AI assistant",
    version="2.0.0"
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


# ============================================================
# JOHN WESLEY KNOWLEDGE BASE
# ============================================================

knowledge = [
    Document(
        page_content="""
        John Wesley is an AI Developer and B.Tech Artificial Intelligence
        and Data Science student at Panimalar Engineering College.
        He is expected to graduate in 2029 and is based in Chennai, India.
        """,
        metadata={"source": "profile"}
    ),

    Document(
        page_content="""
        John's interests include Generative AI, Machine Learning,
        Computer Vision, AI Agents and Open Source.
        """,
        metadata={"source": "interests"}
    ),

    Document(
        page_content="""
        John is currently learning LangGraph, FastAPI, AI Agents,
        Retrieval Augmented Generation (RAG), LLM Engineering and
        System Design for AI Applications.
        """,
        metadata={"source": "learning"}
    ),

    Document(
        page_content="""
        John's programming languages are Python, Java, C and SQL.
        """,
        metadata={"source": "programming"}
    ),

    Document(
        page_content="""
        John's AI and machine learning technologies include OpenCV,
        Scikit-Learn, LangGraph, FastAPI and DeepFace.
        His developer tools include Git, GitHub, VS Code and Jupyter.
        """,
        metadata={"source": "skills"}
    ),

    Document(
        page_content="""
        tiny-langgraph-ai is a beginner-friendly AI assistant built
        with LangGraph and Python. It demonstrates graph-based AI
        workflows and tool integration.
        """,
        metadata={"source": "project_tiny_langgraph_ai"}
    ),

    Document(
        page_content="""
        AI Vision System is a computer vision project using OpenCV
        for real-time object detection.
        """,
        metadata={"source": "project_vision"}
    ),

    Document(
        page_content="""
        AI Face Emotion Recognition is a computer vision project
        using DeepFace and OpenCV to detect facial emotions.
        """,
        metadata={"source": "project_emotion"}
    ),

    Document(
        page_content="""
        AI Object System is an intelligent computer vision project
        focused on object detection.
        """,
        metadata={"source": "project_object"}
    ),

    Document(
        page_content="""
        John's certifications include:
        Google Introduction to Generative AI,
        LinkedIn Learning Generative AI,
        TCS iON AI Foundation,
        Anthropic Claude Code 101,
        AWS Educate Machine Learning Foundations,
        Infosys Springboard Introduction to Artificial Intelligence,
        Infosys Springboard Introduction to Natural Language Processing,
        Infosys Springboard Introduction to Data Science,
        IBM Cybersecurity Fundamentals,
        HP LIFE Introduction to Cybersecurity Awareness,
        HackerRank Problem Solving Intermediate,
        and GUVI C++ Programming for Beginners.
        """,
        metadata={"source": "certifications"}
    ),

    Document(
        page_content="""
        John's career goals include becoming an AI Engineer,
        Machine Learning Engineer, Computer Vision Engineer,
        Open Source Contributor and Research-Oriented Developer.
        """,
        metadata={"source": "career"}
    ),

    Document(
        page_content="""
        John is interested in AI-powered developer tools,
        Agentic AI, real-world AI applications, open source,
        AI internships, student programs and research-oriented projects.
        """,
        metadata={"source": "career_interests"}
    ),

    Document(
        page_content="""
        John's developer philosophy is:
        BUILD, LEARN, EXPERIMENT, FAIL, IMPROVE, SHARE, REPEAT.
        """,
        metadata={"source": "philosophy"}
    )
]


# ============================================================
# EMBEDDINGS
# ============================================================

embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/all-MiniLM-L6-v2"
)


# ============================================================
# VECTOR DATABASE
# ============================================================

vector_store = Chroma(
    collection_name="john_ai_knowledge",
    embedding_function=embeddings,
    persist_directory="./chroma_db"
)


# ============================================================
# LOAD KNOWLEDGE
# ============================================================

def initialize_knowledge_base():
    existing = vector_store.get()

    if len(existing["ids"]) == 0:
        vector_store.add_documents(knowledge)


initialize_knowledge_base()


# ============================================================
# RETRIEVER
# ============================================================

retriever = vector_store.as_retriever(
    search_kwargs={"k": 4}
)


# ============================================================
# LLM
# ============================================================

api_key = os.getenv("OPENAI_API_KEY")

llm = None

if api_key:
    llm = ChatOpenAI(
        model=os.getenv("OPENAI_MODEL", "gpt-4o-mini"),
        temperature=0
    )


# ============================================================
# LANGGRAPH STATE
# ============================================================

class ChatState(TypedDict):
    message: str
    context: str
    response: str


# ============================================================
# RETRIEVAL NODE
# ============================================================

def retrieve_information(state: ChatState):

    question = state["message"]

    documents = retriever.invoke(question)

    if not documents:
        return {
            "message": question,
            "context": "",
            "response": ""
        }

    context = "\n\n".join(
        document.page_content
        for document in documents
    )

    return {
        "message": question,
        "context": context,
        "response": ""
    }


# ============================================================
# AI NODE
# ============================================================

def generate_response(state: ChatState):

    question = state["message"]
    context = state["context"]

    if not context:
        return {
            "message": question,
            "context": context,
            "response": (
                "I don't have that information in John's "
                "current knowledge base."
            )
        }

    if llm is None:
        return {
            "message": question,
            "context": context,
            "response": (
                "Relevant information from John's knowledge base:\n\n"
                + context
            )
        }

    prompt = f"""
You are JOHN AI, the personal AI assistant for John Wesley.

Your job is to answer questions about John Wesley using ONLY
the provided knowledge.

IMPORTANT RULES:

1. Never invent information about John.
2. Never create fake jobs, companies, awards, projects,
   certifications, skills or personal information.
3. If the answer cannot be found in the context, say:
   "I don't have that information in John's current knowledge base."
4. Keep answers clear, natural and useful.
5. Do not mention that you are using a vector database.
6. You may describe John positively, but do not exaggerate
   or invent achievements.

KNOWLEDGE:

{context}

USER QUESTION:

{question}

ANSWER:
"""

    result = llm.invoke(prompt)

    return {
        "message": question,
        "context": context,
        "response": result.content
    }


# ============================================================
# LANGGRAPH
# ============================================================

graph_builder = StateGraph(ChatState)

graph_builder.add_node(
    "retrieve",
    retrieve_information
)

graph_builder.add_node(
    "generate",
    generate_response
)

graph_builder.add_edge(
    START,
    "retrieve"
)

graph_builder.add_edge(
    "retrieve",
    "generate"
)

graph_builder.add_edge(
    "generate",
    END
)

graph = graph_builder.compile()


# ============================================================
# API MODELS
# ============================================================

class ChatRequest(BaseModel):
    message: str


# ============================================================
# ROOT
# ============================================================

@app.get("/")
def root():

    return {
        "name": "JOHN AI",
        "version": "2.0.0",
        "status": "online",
        "system": "RAG + LangGraph"
    }


# ============================================================
# HEALTH
# ============================================================

@app.get("/health")
def health():

    return {
        "status": "healthy",
        "ai": "online",
        "langgraph": "online",
        "rag": "online",
        "vector_database": "online",
        "knowledge_documents": len(knowledge)
    }


# ============================================================
# CHAT
# ============================================================

@app.post("/chat")
def chat(request: ChatRequest):

    if not request.message.strip():

        return {
            "response": "Please enter a question."
        }

    result = graph.invoke({
        "message": request.message,
        "context": "",
        "response": ""
    })

    return {
        "response": result["response"]
    }


# ============================================================
# KNOWLEDGE SEARCH
# ============================================================

@app.get("/search")
def search(q: str):

    if not q.strip():

        return {
            "results": []
        }

    documents = retriever.invoke(q)

    return {
        "query": q,
        "results": [
            {
                "content": document.page_content,
                "source": document.metadata.get("source")
            }
            for document in documents
        ]
    }


# ============================================================
# RUN
# ============================================================

if __name__ == "__main__":

    import uvicorn

    uvicorn.run(
        "main:app",
        host="0.0.0.0",
        port=8000,
        reload=True
    )
    
