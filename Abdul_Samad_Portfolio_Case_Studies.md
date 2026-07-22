# Portfolio Case Studies — Abdul Samad

## Voice Card

**Direct, decision-driven, honest about limits, no hype**

\---

## Bio

I'm Abdul Samad, an AI undergraduate at SZABIST who enjoys building practical software and AI systems that solve real problems. My work focuses on machine learning, automation, full-stack development, and creating tools that people can actually use. I'm always looking for opportunities to learn by building.

\---

## Case Study 1: Clinic Lead Generation System — CodeCelix Internship

**The Problem**
CodeCelix's client needed a steady list of high-quality leads for their clinic outreach, but the process was entirely manual. Someone had to search for businesses one by one, check whether they were still active, verify contact details, remove duplicates, and manually enter everything into a spreadsheet. That process produced around 10–20 verified leads per day and took 2–4 hours of manual work. The client wanted leads their sales team could immediately use for calls, WhatsApp, and email — not outdated or irrelevant contacts that wasted their time.

**What I Did**
I built an automated pipeline: Google Places pulls in candidate businesses matching the client's criteria, n8n (self-hosted on Oracle Cloud) runs the workflow, Groq classifies whether each business is a good fit and filters out duplicates or inactive listings, verified leads get written to a Google Sheet, and WhatsApp notifies the client when a fresh batch is ready for their sales team to review.

I chose to self-host n8n on Oracle Cloud's free tier instead of using n8n Cloud, Zapier, or Make. The project needed to run on free tools, and self-hosting also gave me more control over configuration, credentials, and execution limits than a managed platform would. The tradeoff was extra setup and maintenance — I had to configure and deploy the server myself — but it meant lower long-term cost and no restrictive automation limits.

The biggest challenge was inconsistent data from Google Places — missing phone numbers, outdated websites, duplicate listings. I added validation and filtering steps before saving leads, tuned the Groq prompt when its classifications were inconsistent, added a retry mechanism for low-confidence responses, and added retry/error handling for API timeouts so one failed request wouldn't stop the whole workflow. Building the workflow itself wasn't the hardest part — making it reliable was.

**Outcome**
The workflow runs end to end: collection, classification, verification, spreadsheet updates, and client notification all happen automatically. It's currently in testing and optimization, on track to hit the 100+ verified leads/week target, with my focus right now on lead quality over volume before calling it production-ready. Compared to the old manual process — 10–20 leads/day, 2–4 hours of work — the client's sales team now reviews finished leads instead of collecting and cleaning data themselves.

\---

## Case Study 2: AI Surveillance System — Final Year Project

**The Problem**
Traditional surveillance systems mostly just record video and rely on someone continuously watching camera feeds — inefficient for places like university campuses, office buildings, warehouses, and retail stores, where important events can easily be missed if no one happens to be watching at the right moment. The goal was a system that could automatically detect people and vehicles, track movement across camera feeds, monitor occupancy, and generate real-time alerts instead of leaving footage for someone to review after the fact.

**What I Did**
I used YOLOv8 for detection because it balances accuracy and real-time performance for live video, and DeepSORT for tracking because it maintains consistent IDs across frames through appearance-based re-identification — important for counting people and analyzing movement. I looked at ByteTrack as an alternative but DeepSORT's stable tracking fit the project's goals better. For the backend I chose Flask for its lightweight REST API integration with the inference pipeline, and WebRTC for video delivery since it's built for low-latency streaming — a better fit than plain HTTP video when a dashboard needs to show multiple live feeds at once.

The hardest problem was keeping real-time performance while handling multiple camera streams — running detection and tracking on several feeds simultaneously loads the system heavily, and performance degrades fast as more streams are added. I separated the detection pipeline from the dashboard and optimized frame processing to keep the interface responsive. I also had to tune tracking parameters to reduce duplicate detections and keep object IDs stable when people crossed paths or were briefly occluded.

**Outcome**
The system is a working prototype: a dashboard with user authentication, simulated multi-camera feeds, real-time detection and tracking, occupancy statistics, alert generation, and historical event logging. A production-readiness audit scored it 88/100 — the core functionality is fully implemented and integrated, with the remaining gap being engineering work (scalability testing with more cameras, security hardening, testing under varied conditions, production-grade deployment) rather than missing features. I haven't completed formal benchmarking for FPS across hardware configurations or mAP on a production dataset yet, so I'm not claiming numbers I haven't measured — the system demonstrates real-time detection and tracking on the test setup, and formal evaluation is planned.

\---

## Case Study 3: Loan Status Prediction

**The Problem**
Loan approval decisions depend on many factors — income, loan amount, credit history, education, employment status — and I wanted to build a model that could predict whether a loan application was likely to be approved, working from a structured tabular dataset where each row was an application and the target was approval status.

**What I Did**
I tested a few traditional ML models — Logistic Regression, Decision Trees, and Random Forest — and chose Random Forest as the final model because it handled the tabular data well, captured non-linear relationships, and needed less feature engineering than the alternatives, while giving the most balanced performance on validation data. The hardest part wasn't model selection — it was data preparation: handling missing values, encoding categorical variables, and preprocessing features. That was the biggest lesson from this project — data preparation quality mattered far more to the final result than which algorithm I tried.

**Outcome**
A working ML prototype in Python achieving around 80–85% accuracy on the evaluation dataset. It stayed notebook-based rather than a deployed web app — the goal was demonstrating a complete ML pipeline from preprocessing through prediction, not shipping a product.

\---

## Case Study 4: AI Photo Restoration

**The Problem**
Old or damaged photographs — scratches, fading, noise — need restoration to look clean and presentable, and I wanted to build a tool that could take a damaged image and generate a visibly improved version.

**What I Did**
Rather than training a restoration model from scratch, I integrated existing pre-trained deep learning restoration models into a Python application. I chose this because training state-of-the-art restoration networks from scratch needs large datasets and heavy GPU resources, while pre-trained models delivered strong results with far less development time. The main challenge was that damage varies a lot — some photos just needed denoising, others had severe scratches or missing regions — so restoration quality varied depending on the input. This taught me how much model selection and preprocessing matter in computer vision work, and how to integrate pre-trained models into a practical tool rather than building everything from zero.

**Outcome**
A working desktop prototype: upload a damaged image, get back an enhanced version. It demonstrates the restoration workflow successfully, but I didn't run formal quantitative evaluation (PSNR, SSIM), so I'd call it a functional prototype rather than a benchmarked research project.

\---

## Case Study 5: Smart Budgeting Assistant

**The Problem**
Most people manage personal budgeting through spreadsheets or don't track spending at all. The goal was a desktop app that made tracking income, expenses, and savings goals simple in one place, working from user-entered financial data — income, expense categories, transaction amounts, savings goals.

**What I Did**
I built it in Python with Tkinter for the interface and local data storage. I chose Tkinter because it was lightweight and easy to develop with, without needing to learn a heavier GUI framework — the goal was a complete, usable application, not experimenting with new tech for its own sake. The hardest part was keeping budget calculations, expense tracking, and balance updates consistent as users added, edited, or removed transactions. That taught me the value of separating UI from business logic and validating input carefully to avoid calculation errors.

**Outcome**
A working desktop app: users can record income and expenses, organize transactions into categories, monitor remaining budget, and view basic financial summaries. It's a functional prototype built for learning software development concepts — not deployed online or connected to a cloud database, but it demonstrates solid CRUD operations, local data management, and desktop app development in Python.

\---

## Case Study 6: Portfolio Website

**The Problem**
My projects were scattered across GitHub and my CV, so recruiters or potential clients had to jump between different links to piece together what I'd actually built. I wanted one place where someone could quickly see who I am, what I've worked on, and how to reach me.

**What I Did**
I built it as a single-file HTML site to keep it fast, easy to deploy, and simple to maintain — a full framework felt like unnecessary overhead for a static portfolio. I went with a neural terminal aesthetic and bento-grid layout instead of a generic template because the design itself should reflect my interest in AI and modern web experiences, not just list content. The hardest part wasn't writing the HTML — it was balancing visual effects with usability. Things like the particle canvas background and terminal-inspired UI can easily become distracting or hurt performance if overdone, so I spent real time simplifying animations and improving responsiveness across screen sizes. That taught me a portfolio is about clearly presenting your work, not showing off every visual effect you can build.

**Outcome**
The site is live and publicly accessible, with my projects, skills, education, and GitHub/LinkedIn links. I haven't tracked formal analytics, so I'm not claiming traffic or engagement numbers I don't have. The real outcome is having a professional portfolio I can confidently share with recruiters — it gives a much clearer picture of my work than a CV alone.

\---

## Contact / CTA

Interested in collaborating or discussing AI, automation, or software projects? Feel free to connect with me on LinkedIn or GitHub.

\---

## Before \& After Example

**Generic AI Version**
"Engineered a robust, cutting-edge AI automation pipeline leveraging state-of-the-art technologies to seamlessly generate high-quality leads and drive impactful business outcomes."

**Edited Version (from Case Study 1)**
"Building the workflow itself wasn't the hardest part — making it reliable was. Most of my time went into handling edge cases, validating data, and making sure the automation could recover gracefully instead of failing whenever an external service returned unexpected results."

The first version sounds like it could describe any project by anyone. The second version could only be written by someone who actually built the thing — it names the real difficulty (reliability, not features), admits the boring parts (edge cases, validation), and doesn't inflate what was accomplished.

