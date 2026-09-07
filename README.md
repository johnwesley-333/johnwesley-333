# ============================================================
# JOHN AI — COMPLETE PHASE 1 → PHASE 5
# ONE-FILE PERSONAL AI ASSISTANT
# ============================================================
#
# Run:
#   pip install fastapi uvicorn python-dotenv
#
# Optional real LLM:
#   pip install langchain-openai langgraph
#
# Then:
#   python main.py
#
# Open:
#   http://localhost:8000
#
# ============================================================

import os
import re
import uuid
import html
from datetime import datetime
from typing import Optional

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import HTMLResponse
from pydantic import BaseModel

try:
    from dotenv import load_dotenv
    load_dotenv()
except Exception:
    pass

# Optional LangGraph
try:
    from langgraph.graph import StateGraph, END
    LANGGRAPH = True
except Exception:
    LANGGRAPH = False

# Optional OpenAI
try:
    from langchain_openai import ChatOpenAI
    OPENAI = True
except Exception:
    OPENAI = False


# ============================================================
# CONFIGURATION
# ============================================================

APP_NAME = "JOHN AI"
VERSION = "5.0"

OPENAI_API_KEY = os.getenv("OPENAI_API_KEY", "")
OPENAI_MODEL = os.getenv(
    "OPENAI_MODEL",
    "gpt-4o-mini"
)


# ============================================================
# FASTAPI APPLICATION
# ============================================================

app = FastAPI(
    title=APP_NAME,
    version=VERSION,
    description="JOHN AI Personal AI Assistant"
)

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


# ============================================================
# JOHN AI KNOWLEDGE BASE
# ============================================================

KNOWLEDGE = [

    {
        "title": "John Wesley",
        "category": "profile",
        "source": "john-profile",
        "content": """
John Wesley is an AI Developer and a B.Tech Artificial Intelligence
and Data Science student at Panimalar Engineering College.
His expected graduation year is 2029.
"""
    },

    {
        "title": "Location",
        "category": "profile",
        "source": "john-profile",
        "content": """
John is based in Chennai, India.
"""
    },

    {
        "title": "Education",
        "category": "education",
        "source": "john-profile",
        "content": """
John is pursuing a B.Tech in Artificial Intelligence and Data Science
at Panimalar Engineering College and is expected to graduate in 2029.
"""
    },

    {
        "title": "AI Interests",
        "category": "interests",
        "source": "john-profile",
        "content": """
John's interests include Generative AI, Machine Learning,
Computer Vision, AI Agents, and Open Source.
"""
    },

    {
        "title": "Current Learning",
        "category": "learning",
        "source": "john-profile",
        "content": """
John is currently learning LangGraph, FastAPI, AI Agents,
RAG, LLM Engineering, and System Design for AI Applications.
"""
    },

    {
        "title": "Programming Languages",
        "category": "skills",
        "source": "john-profile",
        "content": """
John works with Python, Java, C, and SQL.
"""
    },

    {
        "title": "AI and ML Technologies",
        "category": "skills",
        "source": "john-profile",
        "content": """
John works with OpenCV, Scikit-Learn, LangGraph, FastAPI,
and DeepFace.
"""
    },

    {
        "title": "Developer Tools",
        "category": "skills",
        "source": "john-profile",
        "content": """
John uses Git, GitHub, VS Code, and Jupyter.
"""
    },

    {
        "title": "Tiny LangGraph AI",
        "category": "projects",
        "source": "github-projects",
        "content": """
tiny-langgraph-ai is a beginner AI assistant project built with
LangGraph and Python. It demonstrates graph workflows and
tool integration.
"""
    },

    {
        "title": "AI Vision System",
        "category": "projects",
        "source": "github-projects",
        "content": """
AI Vision System is a real-time computer vision project
using OpenCV for object detection.
"""
    },

    {
        "title": "AI Face Emotion Recognition",
        "category": "projects",
        "source": "github-projects",
        "content": """
AI Face Emotion Recognition uses DeepFace and OpenCV
for facial emotion recognition.
"""
    },

    {
        "title": "AI Object System",
        "category": "projects",
        "source": "github-projects",
        "content": """
AI Object System is a computer vision project focused
on object detection.
"""
    },

    {
        "title": "Certifications",
        "category": "certifications",
        "source": "john-profile",
        "content": """
John's certifications and learning achievements include
Google Introduction to Generative AI,
LinkedIn Learning Generative AI,
TCS iON AI Foundation,
Anthropic Claude Code 101,
AWS Educate Machine Learning Foundations,
Infosys Springboard AI, NLP and Data Science,
IBM Cybersecurity Fundamentals,
HP LIFE Cybersecurity Awareness,
HackerRank Problem Solving Intermediate,
and GUVI C++ Beginners.
"""
    },

    {
        "title": "Career Goals",
        "category": "career",
        "source": "john-profile",
        "content": """
John's career goals include becoming an AI Engineer,
ML Engineer, Computer Vision Engineer,
Open Source Contributor, and research-oriented developer.
"""
    },

    {
        "title": "Core Values",
        "category": "philosophy",
        "source": "john-profile",
        "content": """
John's core values are:
Build, Learn, Share, Improve, Repeat.
"""
    }

]


# ============================================================
# MEMORY
# ============================================================

MEMORY = {}


def new_session():

    session_id = str(uuid.uuid4())

    MEMORY[session_id] = []

    return session_id


def save_message(session_id, role, content):

    if session_id not in MEMORY:
        MEMORY[session_id] = []

    MEMORY[session_id].append({
        "role": role,
        "content": content,
        "timestamp": datetime.utcnow().isoformat()
    })

    # Prevent unlimited growth
    MEMORY[session_id] = MEMORY[session_id][-30:]


def get_history(session_id):

    return MEMORY.get(session_id, [])


def delete_memory(session_id):

    if session_id in MEMORY:
        del MEMORY[session_id]


# ============================================================
# TEXT SEARCH / RAG
# ============================================================

STOP_WORDS = {
    "the",
    "and",
    "what",
    "who",
    "is",
    "are",
    "does",
    "do",
    "of",
    "to",
    "a",
    "an",
    "in",
    "for",
    "john",
    "tell",
    "me",
    "about",
    "his",
    "he"
}


def tokenize(text):

    words = re.findall(
        r"[a-zA-Z0-9]+",
        text.lower()
    )

    return {
        word
        for word in words
        if word not in STOP_WORDS
    }


def search_knowledge(
    query,
    category=None,
    limit=5
):

    query_words = tokenize(query)

    results = []

    for item in KNOWLEDGE:

        if category and item["category"] != category:
            continue

        searchable = (
            item["title"] + " " +
            item["category"] + " " +
            item["content"]
        )

        words = tokenize(searchable)

        score = len(
            query_words.intersection(words)
        )

        # Bonus for title/category match
        title_words = tokenize(item["title"])

        score += (
            len(query_words.intersection(title_words))
            * 2
        )

        if score > 0:

            results.append({
                "title": item["title"],
                "category": item["category"],
                "source": item["source"],
                "content": item["content"].strip(),
                "score": score
            })

    results.sort(
        key=lambda x: x["score"],
        reverse=True
    )

    return results[:limit]


# ============================================================
# ROUTER
# ============================================================

def route_query(query):

    q = query.lower()

    if any(x in q for x in [
        "project",
        "projects",
        "built",
        "build",
        "github",
        "created"
    ]):
        return "PROJECTS"

    if any(x in q for x in [
        "skill",
        "skills",
        "python",
        "java",
        "c language",
        "sql",
        "opencv",
        "deepface",
        "langgraph",
        "fastapi",
        "technology",
        "technologies"
    ]):
        return "SKILLS"

    if any(x in q for x in [
        "certificate",
        "certification",
        "certifications",
        "course",
        "courses"
    ]):
        return "CERTIFICATIONS"

    if any(x in q for x in [
        "career",
        "career goal",
        "career goals",
        "future",
        "job"
    ]):
        return "CAREER"

    if any(x in q for x in [
        "college",
        "education",
        "study",
        "studies",
        "degree",
        "graduation"
    ]):
        return "EDUCATION"

    if any(x in q for x in [
        "interest",
        "interests",
        "like",
        "passion"
    ]):
        return "INTERESTS"

    if any(x in q for x in [
        "learning",
        "learn",
        "learning now",
        "currently learning"
    ]):
        return "LEARNING"

    if any(x in q for x in [
        "john",
        "who is john",
        "about john",
        "profile",
        "who are you"
    ]):
        return "ABOUT_JOHN"

    if any(x in q for x in [
        "hello",
        "hi",
        "hey",
        "good morning",
        "good evening"
    ]):
        return "GENERAL"

    return "GENERAL"


# ============================================================
# CATEGORY RETRIEVAL
# ============================================================

CATEGORY_MAP = {

    "PROJECTS": "projects",

    "SKILLS": "skills",

    "CERTIFICATIONS": "certifications",

    "CAREER": "career",

    "EDUCATION": "education",

    "INTERESTS": "interests",

    "LEARNING": "learning",

    "ABOUT_JOHN": None,

    "GENERAL": None
}


def retrieve(query, route):

    category = CATEGORY_MAP.get(route)

    return search_knowledge(
        query,
        category=category,
        limit=5
    )


# ============================================================
# CONTEXT
# ============================================================

def make_context(results):

    if not results:
        return ""

    blocks = []

    for result in results:

        blocks.append(
            f"""
TITLE: {result['title']}
CATEGORY: {result['category']}
SOURCE: {result['source']}
CONTENT:
{result['content']}
"""
        )

    return "\n".join(blocks)


# ============================================================
# PERSONAL TOOLS
# ============================================================

def get_projects():

    return [
        x for x in KNOWLEDGE
        if x["category"] == "projects"
    ]


def get_skills():

    return [
        x for x in KNOWLEDGE
        if x["category"] == "skills"
    ]


def get_certifications():

    return [
        x for x in KNOWLEDGE
        if x["category"] == "certifications"
    ]


def get_profile():

    return [
        x for x in KNOWLEDGE
        if x["category"] == "profile"
    ]


# ============================================================
# LOCAL AI RESPONSE
# ============================================================

def local_response(
    query,
    results,
    route
):

    q = query.lower()

    # Greeting
    if route == "GENERAL" and any(
        x in q
        for x in [
            "hello",
            "hi",
            "hey",
            "good morning",
            "good evening"
        ]
    ):

        return (
            "Hey! 👋 I'm JOHN AI.\n\n"
            "I'm John's personal AI assistant. "
            "You can ask me about his projects, skills, "
            "certifications, education, interests, "
            "learning journey, and career goals."
        )

    # Capabilities
    if "what can you do" in q:

        return (
            "I can help you explore John's:\n\n"
            "• 🤖 AI projects\n"
            "• 💻 Technical skills\n"
            "• 🎓 Education\n"
            "• 🏆 Certifications\n"
            "• 🚀 Career goals\n"
            "• 🧠 Current learning\n"
            "• 🔬 AI interests\n\n"
            "I can also remember our conversation "
            "during the current session."
        )

    # Projects
    if route == "PROJECTS":

        projects = get_projects()

        answer = "🚀 JOHN'S PROJECTS\n\n"

        for project in projects:

            answer += (
                f"• {project['title']}\n"
                f"  {project['content'].strip()}\n\n"
            )

        return answer.strip()

    # Skills
    if route == "SKILLS":

        return (
            "💻 JOHN'S TECHNICAL SKILLS\n\n"
            "Programming:\n"
            "• Python\n"
            "• Java\n"
            "• C\n"
            "• SQL\n\n"
            "AI / ML:\n"
            "• OpenCV\n"
            "• Scikit-Learn\n"
            "• DeepFace\n"
            "• LangGraph\n"
            "• FastAPI\n\n"
            "Tools:\n"
            "• Git\n"
            "• GitHub\n"
            "• VS Code\n"
            "• Jupyter"
        )

    # Certifications
    if route == "CERTIFICATIONS":

        certs = get_certifications()

        return (
            "🏆 JOHN'S CERTIFICATIONS\n\n" +
            certs[0]["content"].strip()
        )

    # Career
    if route == "CAREER":

        results = search_knowledge(
            "career goals",
            category="career"
        )

        if results:
            return (
                "🚀 CAREER GOALS\n\n" +
                results[0]["content"]
            )

    # Education
    if route == "EDUCATION":

        results = search_knowledge(
            "education college degree graduation",
            category="education"
        )

        if results:
            return (
                "🎓 EDUCATION\n\n" +
                results[0]["content"]
            )

    # Interests
    if route == "INTERESTS":

        results = search_knowledge(
            "AI interests",
            category="interests"
        )

        if results:
            return (
                "🔬 AI INTERESTS\n\n" +
                results[0]["content"]
            )

    # Learning
    if route == "LEARNING":

        results = search_knowledge(
            "currently learning",
            category="learning"
        )

        if results:
            return (
                "🧠 CURRENTLY LEARNING\n\n" +
                results[0]["content"]
            )

    # About
    if route == "ABOUT_JOHN":

        return (
            "👤 ABOUT JOHN\n\n"
            "John Wesley is an AI Developer and a "
            "B.Tech Artificial Intelligence and Data Science "
            "student at Panimalar Engineering College.\n\n"
            "His interests include Generative AI, Machine "
            "Learning, Computer Vision, AI Agents, and "
            "Open Source."
        )

    # RAG answer
    if results:

        answer = (
            "🔎 KNOWLEDGE BASE RESULT\n\n"
        )

        for result in results[:3]:

            answer += (
                f"**{result['title']}**\n"
                f"{result['content']}\n\n"
            )

        return answer.strip()

    # Unknown
    return (
        "I don't have enough information in my "
        "knowledge base to answer that accurately."
    )


# ============================================================
# OPENAI RESPONSE
# ============================================================

def openai_response(
    query,
    context,
    history
):

    if not OPENAI:
        return None

    if not OPENAI_API_KEY:
        return None

    try:

        model = ChatOpenAI(
            model=OPENAI_MODEL,
            temperature=0.3,
            api_key=OPENAI_API_KEY
        )

        system_prompt = """
You are JOHN AI.

You are the personal AI assistant for John Wesley.

Your job is to answer questions about John using the
provided knowledge base and conversation history.

IMPORTANT RULES:

1. Never invent information about John.
2. Never fabricate projects.
3. Never fabricate certificates.
4. Never fabricate jobs.
5. Never fabricate awards.
6. Never fabricate companies.
7. Never fabricate experience.
8. Never fabricate skills.
9. If information is unavailable, say:
   "I don't have that information in my knowledge base."
10. Be natural and conversational.
11. Keep answers useful and reasonably concise.
12. Use the retrieved context as the source of truth.
13. Do not mention internal implementation details unless asked.
"""

        messages = [
            (
                "system",
                system_prompt
            )
        ]

        for message in history[-10:]:

            role = message["role"]

            if role == "user":

                messages.append(
                    (
                        "human",
                        message["content"]
                    )
                )

            elif role == "assistant":

                messages.append(
                    (
                        "assistant",
                        message["content"]
                    )
                )

        prompt = f"""
RETRIEVED KNOWLEDGE:

{context if context else "No relevant knowledge was found."}

CURRENT USER QUESTION:

{query}
"""

        messages.append(
            (
                "human",
                prompt
            )
        )

        result = model.invoke(
            messages
        )

        return result.content

    except Exception as error:

        print(
            "OpenAI error:",
            error
        )

        return None


# ============================================================
# LANGGRAPH WORKFLOW
# ============================================================

if LANGGRAPH:

    class AIState(dict):
        pass

    def graph_router(state):

        return {
            "route": route_query(
                state["query"]
            )
        }

    def graph_retrieve(state):

        results = retrieve(
            state["query"],
            state["route"]
        )

        return {
            "results": results,
            "context": make_context(results)
        }

    def graph_generate(state):

        history = get_history(
            state["session_id"]
        )

        response = openai_response(
            state["query"],
            state.get("context", ""),
            history
        )

        if not response:

            response = local_response(
                state["query"],
                state.get("results", []),
                state["route"]
            )

        return {
            "response": response
        }

    workflow = StateGraph(dict)

    workflow.add_node(
        "router",
        graph_router
    )

    workflow.add_node(
        "retriever",
        graph_retrieve
    )

    workflow.add_node(
        "generator",
        graph_generate
    )

    workflow.set_entry_point(
        "router"
    )

    workflow.add_edge(
        "router",
        "retriever"
    )

    workflow.add_edge(
        "retriever",
        "generator"
    )

    workflow.add_edge(
        "generator",
        END
    )

    AI_GRAPH = workflow.compile()

else:

    AI_GRAPH = None


# ============================================================
# API MODELS
# ============================================================

class ChatRequest(BaseModel):

    message: str

    session_id: Optional[str] = None


class ChatResponse(BaseModel):

    response: str

    session_id: str

    route: str

    sources: list


# ============================================================
# HEALTH
# ============================================================

@app.get("/health")
def health():

    return {
        "status": "online",
        "name": APP_NAME,
        "version": VERSION,
        "langgraph": LANGGRAPH,
        "openai": bool(
            OPENAI and OPENAI_API_KEY
        ),
        "knowledge_items": len(KNOWLEDGE),
        "sessions": len(MEMORY)
    }


# ============================================================
# ROOT
# ============================================================

@app.get("/api")
def api_root():

    return {
        "name": APP_NAME,
        "version": VERSION,
        "status": "online",
        "message": "JOHN AI is running."
    }


# ============================================================
# CREATE SESSION
# ============================================================

@app.post("/session")
def create_session():

    session_id = new_session()

    return {
        "session_id": session_id
    }


# ============================================================
# CHAT
# ============================================================

@app.post(
    "/chat",
    response_model=ChatResponse
)
def chat(request: ChatRequest):

    message = request.message.strip()

    if not message:

        session_id = (
            request.session_id
            or new_session()
        )

        return ChatResponse(
            response="Please enter a message.",
            session_id=session_id,
            route="GENERAL",
            sources=[]
        )

    session_id = (
        request.session_id
        or new_session()
    )

    if session_id not in MEMORY:
        MEMORY[session_id] = []

    save_message(
        session_id,
        "user",
        message
    )

    route = route_query(
        message
    )

    sources = []

    if AI_GRAPH:

        result = AI_GRAPH.invoke({

            "query": message,

            "session_id": session_id

        })

        response = result.get(
            "response",
            ""
        )

        for item in result.get(
            "results",
            []
        ):

            sources.append(
                item["title"]
            )

    else:

        results = retrieve(
            message,
            route
        )

        response = openai_response(
            message,
            make_context(results),
            get_history(session_id)
        )

        if not response:

            response = local_response(
                message,
                results,
                route
            )

        sources = [
            item["title"]
            for item in results
        ]

    save_message(
        session_id,
        "assistant",
        response
    )

    return ChatResponse(
        response=response,
        session_id=session_id,
        route=route,
        sources=sources
    )


# ============================================================
# HISTORY
# ============================================================

@app.get(
    "/history/{session_id}"
)
def history(session_id: str):

    return {
        "session_id": session_id,
        "messages": get_history(
            session_id
        )
    }


# ============================================================
# DELETE HISTORY
# ============================================================

@app.delete(
    "/history/{session_id}"
)
def clear_history(session_id: str):

    delete_memory(
        session_id
    )

    return {
        "success": True,
        "message": "Conversation memory cleared."
    }


# ============================================================
# SEARCH API
# ============================================================

@app.get("/search")
def search(
    q: str,
    category: Optional[str] = None
):

    results = search_knowledge(
        q,
        category=category
    )

    return {
        "query": q,
        "category": category,
        "results": results
    }


# ============================================================
# PROJECT API
# ============================================================

@app.get("/projects")
def projects():

    return {
        "projects": get_projects()
    }


# ============================================================
# SKILLS API
# ============================================================

@app.get("/skills")
def skills():

    return {
        "skills": get_skills()
    }


# ============================================================
# CERTIFICATIONS API
# ============================================================

@app.get("/certifications")
def certifications():

    return {
        "certifications": get_certifications()
    }


# ============================================================
# PROFILE API
# ============================================================

@app.get("/profile")
def profile():

    return {
        "profile": get_profile()
    }


# ============================================================
# FRONTEND
# ============================================================

HTML = r"""
<!DOCTYPE html>

<html lang="en">

<head>

<meta charset="UTF-8">

<meta
name="viewport"
content="width=device-width, initial-scale=1.0"
>

<title>JOHN AI — Personal AI Assistant</title>

<style>

* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

html {
    scroll-behavior: smooth;
}

body {

    background:
        radial-gradient(
            circle at 20% 20%,
            rgba(0,255,255,.10),
            transparent 30%
        ),
        radial-gradient(
            circle at 80% 80%,
            rgba(0,100,255,.10),
            transparent 30%
        ),
        #03070b;

    color: #dffcff;

    font-family:
        Inter,
        Arial,
        sans-serif;

    min-height: 100vh;
}

body::before {

    content: "";

    position: fixed;

    inset: 0;

    pointer-events: none;

    background-image:
        linear-gradient(
            rgba(0,255,255,.025) 1px,
            transparent 1px
        ),
        linear-gradient(
            90deg,
            rgba(0,255,255,.025) 1px,
            transparent 1px
        );

    background-size: 40px 40px;

}

header {

    position: sticky;

    top: 0;

    z-index: 100;

    backdrop-filter: blur(20px);

    background: rgba(3,7,11,.80);

    border-bottom:
        1px solid rgba(0,255,255,.15);

}

nav {

    max-width: 1200px;

    margin: auto;

    padding: 18px 25px;

    display: flex;

    align-items: center;

    justify-content: space-between;

}

.logo {

    font-size: 22px;

    font-weight: 900;

    letter-spacing: 4px;

    color: #00f6ff;

    text-shadow:
        0 0 15px rgba(0,246,255,.7);

}

.status {

    display: flex;

    align-items: center;

    gap: 8px;

    font-size: 12px;

    letter-spacing: 2px;

}

.status-dot {

    width: 9px;

    height: 9px;

    border-radius: 50%;

    background: #ff4040;

    box-shadow:
        0 0 12px #ff4040;

}

.status-dot.online {

    background: #00ff9d;

    box-shadow:
        0 0 15px #00ff9d;

}

.hero {

    max-width: 1200px;

    margin: auto;

    padding:
        90px 25px
        50px;

    text-align: center;

}

.badge {

    display: inline-block;

    padding: 8px 15px;

    border:
        1px solid rgba(0,246,255,.4);

    border-radius: 30px;

    color: #00f6ff;

    font-size: 11px;

    letter-spacing: 3px;

    margin-bottom: 25px;

}

h1 {

    font-size:
        clamp(55px, 12vw, 140px);

    line-height: .9;

    letter-spacing: -7px;

    color: white;

    text-shadow:
        0 0 30px rgba(0,246,255,.3);

}

.hero span {

    color: #00f6ff;

}

.subtitle {

    max-width: 650px;

    margin:
        30px auto;

    color: #8ca8ad;

    font-size: 17px;

    line-height: 1.8;

}

.container {

    max-width: 1100px;

    margin: auto;

    padding: 20px;

}

.panel {

    background:
        rgba(7,17,22,.82);

    border:
        1px solid rgba(0,246,255,.15);

    border-radius: 20px;

    overflow: hidden;

    box-shadow:
        0 20px 80px
        rgba(0,0,0,.35);

}

.panel-head {

    padding: 18px 22px;

    border-bottom:
        1px solid rgba(0,246,255,.12);

    display: flex;

    justify-content: space-between;

    align-items: center;

}

.panel-title {

    font-size: 12px;

    letter-spacing: 3px;

    color: #00f6ff;

}

.chat {

    height: 500px;

    overflow-y: auto;

    padding: 25px;

}

.message {

    margin-bottom: 22px;

    max-width: 85%;

}

.message.user {

    margin-left: auto;

}

.message.ai {

    margin-right: auto;

}

.message-label {

    font-size: 10px;

    letter-spacing: 2px;

    color: #00f6ff;

    margin-bottom: 7px;

}

.bubble {

    padding: 15px 18px;

    border-radius: 15px;

    line-height: 1.7;

    white-space: pre-wrap;

}

.user .bubble {

    background:
        rgba(0,246,255,.10);

    border:
        1px solid rgba(0,246,255,.25);

}

.ai .bubble {

    background:
        rgba(255,255,255,.035);

    border:
        1px solid rgba(255,255,255,.08);

}

.composer {

    padding: 20px;

    border-top:
        1px solid rgba(0,246,255,.12);

    display: flex;

    gap: 10px;

}

textarea {

    flex: 1;

    resize: none;

    min-height: 55px;

    max-height: 150px;

    background:
        #020609;

    color: white;

    border:
        1px solid rgba(0,246,255,.20);

    border-radius: 12px;

    padding: 16px;

    outline: none;

    font-size: 15px;

}

textarea:focus {

    border-color: #00f6ff;

    box-shadow:
        0 0 20px
        rgba(0,246,255,.08);

}

button {

    border: none;

    cursor: pointer;

    border-radius: 12px;

    padding: 0 20px;

    background: #00f6ff;

    color: #001014;

    font-weight: 900;

    letter-spacing: 1px;

}

button:hover {

    box-shadow:
        0 0 25px
        rgba(0,246,255,.45);

}

.tools {

    max-width: 1100px;

    margin: 20px auto;

    padding: 0 20px;

    display: flex;

    gap: 10px;

    flex-wrap: wrap;

}

.tool {

    background:
        rgba(0,246,255,.05);

    border:
        1px solid rgba(0,246,255,.15);

    color: #9bdde2;

    padding: 10px 15px;

    border-radius: 30px;

    cursor: pointer;

    font-size: 12px;

}

.tool:hover {

    border-color: #00f6ff;

    color: #00f6ff;

}

.grid {

    max-width: 1100px;

    margin: 50px auto;

    padding: 20px;

    display: grid;

    grid-template-columns:
        repeat(
            auto-fit,
            minmax(220px, 1fr)
        );

    gap: 15px;

}

.card {

    background:
        rgba(255,255,255,.025);

    border:
        1px solid rgba(0,246,255,.12);

    border-radius: 16px;

    padding: 25px;

    transition: .3s;

}

.card:hover {

    transform: translateY(-5px);

    border-color:
        rgba(0,246,255,.45);

    box-shadow:
        0 15px 40px
        rgba(0,246,255,.06);

}

.card h3 {

    color: #00f6ff;

    margin-bottom: 12px;

}

.card p {

    color: #8ca8ad;

    line-height: 1.7;

    font-size: 14px;

}

footer {

    text-align: center;

    padding: 50px;

    color: #526b70;

    font-size: 12px;

    letter-spacing: 2px;

}

.typing {

    display: inline-flex;

    gap: 5px;

}

.typing i {

    width: 5px;

    height: 5px;

    background: #00f6ff;

    border-radius: 50%;

    animation:
        pulse 1s infinite;

}

.typing i:nth-child(2) {

    animation-delay: .15s;

}

.typing i:nth-child(3) {

    animation-delay: .3s;

}

@keyframes pulse {

    0%,100% {
        opacity: .2;
        transform: translateY(0);
    }

    50% {
        opacity: 1;
        transform: translateY(-4px);
    }

}

@media(max-width:700px) {

    .composer {
        flex-direction: column;
    }

    button {
        height: 50px;
    }

    h1 {
        letter-spacing: -3px;
    }

}

</style>

</head>

<body>

<header>

<nav>

<div class="logo">
JOHN AI
</div>

<div class="status">

<div
id="statusDot"
class="status-dot">
</div>

<span id="statusText">
CONNECTING
</span>

</div>

</nav>

</header>


<section class="hero">

<div class="badge">
PERSONAL AI ASSISTANT
</div>

<h1>
JOHN <span>AI</span>
</h1>

<p class="subtitle">

An intelligent personal AI system built with
FastAPI, LangGraph, RAG, memory, and LLM technology.

</p>

</section>


<div class="tools">

<div
class="tool"
onclick="ask('Tell me about John')">
ABOUT JOHN
</div>

<div
class="tool"
onclick="ask('What projects has John built?')">
PROJECTS
</div>

<div
class="tool"
onclick="ask('What are John skills?')">
SKILLS
</div>

<div
class="tool"
onclick="ask('What certifications does John have?')">
CERTIFICATIONS
</div>

<div
class="tool"
onclick="ask('What are Johns career goals?')">
CAREER
</div>

<div
class="tool"
onclick="ask('What is John currently learning?')">
LEARNING
</div>

<div
class="tool"
onclick="newChat()">
NEW CHAT
</div>

<div
class="tool"
onclick="clearChat()">
CLEAR CHAT
</div>

</div>


<div class="container">

<div class="panel">

<div class="panel-head">

<div class="panel-title">
JOHN AI // NEURAL INTERFACE
</div>

<div
id="route"
style="
font-size:10px;
color:#58777c;
letter-spacing:1px;
">
READY
</div>

</div>


<div
id="chat"
class="chat">

<div class="message ai">

<div class="message-label">
JOHN AI
</div>

<div class="bubble">

Hello 👋

I'm JOHN AI — John's personal AI assistant.

Ask me anything about his projects,
skills, education, certifications,
interests, learning journey, or career goals.

</div>

</div>

</div>


<div class="composer">

<textarea
id="input"
placeholder="Ask JOHN AI something..."
onkeydown="handleKey(event)"
></textarea>

<button
onclick="sendMessage()">
SEND
</button>

</div>

</div>

</div>


<div class="grid">

<div class="card">

<h3>🤖 AI ENGINE</h3>

<p>
FastAPI backend with optional LLM integration
and intelligent response generation.
</p>

</div>

<div class="card">

<h3>🧠 MEMORY</h3>

<p>
Session-based conversation memory allows
JOHN AI to understand the current conversation.
</p>

</div>

<div class="card">

<h3>🔎 RAG</h3>

<p>
Knowledge retrieval finds relevant information
before generating answers.
</p>

</div>

<div class="card">

<h3>🕸️ LANGGRAPH</h3>

<p>
Queries are routed through a structured
AI workflow when LangGraph is installed.
</p>

</div>

</div>


<footer>

JOHN AI // BUILD • LEARN • SHARE • IMPROVE • REPEAT

</footer>


<script>

const API = "";

let sessionId =
    localStorage.getItem(
        "john_ai_session"
    );


const chat =
    document.getElementById(
        "chat"
    );

const input =
    document.getElementById(
        "input"
    );

const route =
    document.getElementById(
        "route"
    );

const statusDot =
    document.getElementById(
        "statusDot"
    );

const statusText =
    document.getElementById(
        "statusText"
    );


async function checkHealth() {

    try {

        const response =
            await fetch(
                API + "/health"
            );

        if (!response.ok)
            throw new Error();

        statusDot.classList.add(
            "online"
        );

        statusText.textContent =
            "ONLINE";

    }

    catch {

        statusDot.classList.remove(
            "online"
        );

        statusText.textContent =
            "OFFLINE";

    }

}


function addMessage(
    type,
    text
) {

    const wrapper =
        document.createElement(
            "div"
        );

    wrapper.className =
        "message " + type;

    const label =
        document.createElement(
            "div"
        );

    label.className =
        "message-label";

    label.textContent =
        type === "user"
        ? "YOU"
        : "JOHN AI";

    const bubble =
        document.createElement(
            "div"
        );

    bubble.className =
        "bubble";

    bubble.textContent =
        text;

    wrapper.appendChild(
        label
    );

    wrapper.appendChild(
        bubble
    );

    chat.appendChild(
        wrapper
    );

    chat.scrollTop =
        chat.scrollHeight;

}


function addTyping() {

    const wrapper =
        document.createElement(
            "div"
        );

    wrapper.id =
        "typingMessage";

    wrapper.className =
        "message ai";

    wrapper.innerHTML = `

        <div class="message-label">
            JOHN AI
        </div>

        <div class="bubble">

            <span class="typing">
                <i></i>
                <i></i>
                <i></i>
            </span>

        </div>

    `;

    chat.appendChild(
        wrapper
    );

    chat.scrollTop =
        chat.scrollHeight;

}


function removeTyping() {

    const element =
        document.getElementById(
            "typingMessage"
        );

    if (element)
        element.remove();

}


async function sendMessage() {

    const message =
        input.value.trim();

    if (!message)
        return;

    addMessage(
        "user",
        message
    );

    input.value = "";

    addTyping();

    try {

        const response =
            await fetch(
                API + "/chat",
                {
                    method: "POST",

                    headers: {
                        "Content-Type":
                            "application/json"
                    },

                    body: JSON.stringify({

                        message:
                            message,

                        session_id:
                            sessionId

                    })

                }
            );

        if (!response.ok)
            throw new Error(
                "Request failed"
            );

        const data =
            await response.json();

        sessionId =
            data.session_id;

        localStorage.setItem(
            "john_ai_session",
            sessionId
        );

        removeTyping();

        addMessage(
            "ai",
            data.response
        );

        route.textContent =
            data.route || "GENERAL";

    }

    catch(error) {

        removeTyping();

        addMessage(
            "ai",
            "⚠️ I couldn't connect to the JOHN AI backend. Make sure the server is running."
        );

    }

}


function ask(question) {

    input.value =
        question;

    sendMessage();

}


function handleKey(event) {

    if (
        event.key === "Enter" &&
        !event.shiftKey
    ) {

        event.preventDefault();

        sendMessage();

    }

}


function newChat() {

    sessionId = null;

    localStorage.removeItem(
        "john_ai_session"
    );

    chat.innerHTML = "";

    addMessage(
        "ai",
        "New session created. 👋 What would you like to know?"
    );

    route.textContent =
        "READY";

}


async function clearChat() {

    if (sessionId) {

        try {

            await fetch(
                API +
                "/history/" +
                sessionId,
                {
                    method:
                        "DELETE"
                }
            );

        }

        catch {}

    }

    chat.innerHTML = "";

    addMessage(
        "ai",
        "Conversation memory cleared. 🧠"
    );

}


checkHealth();

setInterval(
    checkHealth,
    10000
);

</script>

</body>

</html>
"""


# ============================================================
# SERVE FRONTEND
# ============================================================

@app.get(
    "/",
    response_class=HTMLResponse
)
def frontend():

    return HTML


# ============================================================
# START SERVER
# ============================================================

if __name__ == "__main__":

    import uvicorn

    print()
    print("=" * 60)
    print("                 JOHN AI")
    print("          PERSONAL AI ASSISTANT")
    print("=" * 60)
    print()
    print(
        "Website : http://localhost:8000"
    )
    print(
        "API     : http://localhost:8000/api"
    )
    print(
        "Docs    : http://localhost:8000/docs"
    )
    print()
    print(
        "Knowledge items:",
        len(KNOWLEDGE)
    )
    print(
        "LangGraph:",
        LANGGRAPH
    )
    print(
        "OpenAI:",
        bool(
            OPENAI and OPENAI_API_KEY
        )
    )
    print()
    print("=" * 60)
    print()

    uvicorn.run(
        app,
        host="0.0.0.0",
        port=8000,
        reload=False
    )
