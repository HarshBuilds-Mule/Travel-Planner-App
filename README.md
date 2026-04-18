# Travel-Planner-App

## 📝 Description

Travel-Planner-App is a modern application designed to simplify trip planning and itinerary management. It enables users to organize travel details, visualize routes, and leverage intelligent insights for a smarter travel experience.

The system is built using **MuleSoft** for API-led connectivity, **SvelteKit** for the frontend, and **MySQL** for persistent data storage. It also integrates **AI services** to enhance planning, recommendations, and decision-making.

---

## ✨ Features

* Interactive map-based travel planning
* Route and location integration
* Trip and itinerary management
* 🤖 AI-powered travel recommendations
* 🤖 Smart itinerary suggestions and optimization
* API-led architecture using MuleSoft
* Scalable and modular design
* Secure data storage with MySQL

---

## 🛠️ Tech Stack

* **MuleSoft** – API integration and backend services
* **SvelteKit** – Frontend framework
* **MySQL** – Database
* **AI Integration** – Intelligent recommendations & automation

---

## 🧩 Architecture (MuleSoft API-Led + AI)

```id="arch2"
          ┌──────────────────────────────┐
          │     Experience API Layer     │
          │      (SvelteKit UI)          │
          └──────────────┬───────────────┘
                         │
          ┌──────────────▼───────────────┐
          │       Process API Layer      │
          │  - Trip Planning Logic       │
          │  - AI Orchestration         │
          └──────────────┬───────────────┘
                         │
     ┌───────────────────▼───────────────────┐
     │           System API Layer            │
     │  - Geo/Maps APIs                     │
     │  - Weather APIs                      │
     │  - Routing APIs                      │
     │  - AI Services (LLM / External APIs) │
     │  - MySQL Database                    │
     └──────────────────────────────────────┘
```

---

## 🔍 Architecture Explanation

* **Experience APIs**
  Handle frontend interactions via **SvelteKit**, delivering user-friendly travel planning interfaces.

* **Process APIs**
  Manage business logic and coordinate AI-driven features such as:

  * Generating travel suggestions
  * Optimizing itineraries
  * Aggregating data from multiple services

* **System APIs**
  Connect to:

  * External services (maps, weather, routing)
  * AI services (for intelligent recommendations)
  * **MySQL** database for persistent storage

This design ensures:

* Clean separation of concerns
* Scalable AI integration without tightly coupling components
* Reusable APIs across different platforms

---

## ⚡ Quick Start

```bash id="quick2"
# Clone the repository
git clone https://github.com/HarshBuilds-Mule/Travel-Planner-App.git

# Navigate to frontend
cd frontend

# Install dependencies
npm install

# Run development server
npm run dev
```

---

## 🚀 Run Commands

* `npm run dev` – Start development server
* `npm run build` – Build for production
* `npm run preview` – Preview production build

---

## 📁 Project Structure

```id="struct2"
.
├── Mule-code-jars   # Prebuilt MuleSoft APIs
├── mule-code        # MuleSoft source projects
└── frontend         # SvelteKit frontend
```

---

## 👥 Contributing

Contributions are welcome:

1. Fork the repository
2. Create a new branch (`feature/your-feature`)
3. Commit your changes
4. Push to your branch
5. Open a pull request

---
