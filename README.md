<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Smart Farming AI Assistant</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      theme: {
        extend: {
          animation: {
            'gradient': 'gradient 15s ease infinite',
          },
          keyframes: {
            gradient: {
              '0%, 100%': { backgroundPosition: '0% 50%' },
              '50%': { backgroundPosition: '100% 50%' },
            }
          }
        }
      }
    }
  </script>
  <style>
    .scrollbar-hide::-webkit-scrollbar { display: none; }
    .scrollbar-hide { -ms-overflow-style: none; scrollbar-width: none; }
    .gradient-bg {
      background: linear-gradient(-45deg, #ee7752, #e73c7e, #23a6d5, #23d5ab);
      background-size: 400% 400%;
      animation: gradient 15s ease infinite;
    }
  </style>
</head>
<body class="gradient-bg min-h-screen">
  <div id="root"></div>

  <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <script src="https://unpkg.com/lucide-react@0.263.1/dist/umd/lucide.min.js"></script>

  <script type="text/babel">
    const { Camera, Sparkles, Video, Music, Globe, Search, Leaf, Menu, Send, Upload, User, Bot, X, Plus } = lucide;

    const languages = {
      english: { name: "English", code: "en" },
      tamil: { name: "தமிழ்", code: "ta" },
      hindi: { name: "हिंदी", code: "hi" },
      telugu: { name: "తెలుగు", code: "te" },
      kannada: { name: "ಕನ್ನಡ", code: "kn" },
      malayalam: { name: "മലയാളം", code: "ml" },
    };

    const cropDatabase = {
      Rice: {
        names: { en: "Rice", hi: "चावल", ta: "அரிசி", te: "బియ్యం", kn: "ಅಕ್ಕಿ", ml: "അരി" },
        season: "Kharif/Rabi",
        compatible: ["Green Gram", "Black Gram"],
        incompatible: ["Wheat"],
        category: "cereal",
      },
      Wheat: {
        names: { en: "Wheat", hi: "गेहूं", ta: "கோதுமை", te: "గోధుమ", kn: "ಗೋಧಿ", ml: "ഗോതമ്പ്" },
        season: "Rabi",
        compatible: ["Mustard", "Gram"],
        incompatible: ["Rice"],
        category: "cereal",
      },
      Maize: {
        names: { en: "Maize", hi: "मक्का", ta: "மக்காச்சோளம்", te: "మొక్కజొన్న", kn: "ಜೋಳ", ml: "ചോളം" },
        season: "Kharif/Rabi",
        compatible: ["Beans", "Pumpkin"],
        incompatible: [],
        category: "cereal",
      },
      Tomato: {
        names: { en: "Tomato", hi: "टमाटर", ta: "தக்காளி", te: "టమాటో", kn: "ಟೊಮೇಟೊ", ml: "തക്കാളി" },
        season: "Rabi/Summer",
        compatible: ["Basil", "Marigold"],
        incompatible: ["Potato"],
        category: "vegetable",
      },
      Cotton: {
        names: { en: "Cotton", hi: "कपास", ta: "பருத்தி", te: "పత్తి", kn: "ಹತ್ತಿ", ml: "പരുത്തി" },
        season: "Kharif",
        compatible: ["Marigold", "Chili"],
        incompatible: [],
        category: "fiber_crop",
      },
      Sugarcane: {
        names: { en: "Sugarcane", hi: "गन्ना", ta: "கரும்பு", te: "చెరకు", kn: "ಕಬ್ಬು", ml: "കരിമ്പു" },
        season: "Annual",
        compatible: ["Potato", "Onion"],
        incompatible: ["Wheat"],
        category: "cash_crop",
      },
    };

    const aiModules = [
      {
        id: "image-analysis",
        name: "Field Image Analysis",
        icon: Camera,
        model: "microsoft/resnet-50",
        description: "Analyze field images for disease detection",
        colorScheme: { gradient: "from-pink-500 to-rose-500", bg: "bg-pink-500/20", border: "border-pink-400/40" },
      },
      {
        id: "crop-advisor",
        name: "Intelligent Crop Advisor",
        icon: Sparkles,
        model: "microsoft/DialoGPT",
        description: "Get farming advice and recommendations",
        colorScheme: { gradient: "from-amber-500 to-orange-500", bg: "bg-amber-500/20", border: "border-amber-400/40" },
      },
      {
        id: "video-guide",
        name: "Video Farming Guides",
        icon: Video,
        model: "Video Generation AI",
        description: "Visual farming demonstrations",
        colorScheme: { gradient: "from-cyan-500 to-sky-500", bg: "bg-cyan-500/20", border: "border-cyan-400/40" },
      },
      {
        id: "voice-assistant",
        name: "Voice Assistant",
        icon: Music,
        model: "suno/bark",
        description: "Listen to advice in your language",
        colorScheme: { gradient: "from-emerald-500 to-green-500", bg: "bg-emerald-500/20", border: "border-emerald-400/40" },
      },
      {
        id: "text-translator",
        name: "Multilingual Translator",
        icon: Globe,
        model: "Helsinki-NLP/opus-mt",
        description: "Translate farming content",
        colorScheme: { gradient: "from-violet-500 to-purple-500", bg: "bg-violet-500/20", border: "border-violet-400/40" },
      },
      {
        id: "visual-search",
        name: "Visual Crop Search",
        icon: Search,
        model: "openai/clip-vit",
        description: "Search crops using images",
        colorScheme: { gradient: "from-fuchsia-500 to-pink-500", bg: "bg-fuchsia-500/20", border: "border-fuchsia-400/40" },
      },
    ];

    function SmartFarmingApp() {
      const [selectedLanguage, setSelectedLanguage] = React.useState("english");
      const [selectedCrops, setSelectedCrops] = React.useState([]);
      const [activeModule, setActiveModule] = React.useState(null);
      const [isSidebarOpen, setIsSidebarOpen] = React.useState(true);
      const [messages, setMessages] = React.useState({});
      const [isProcessing, setIsProcessing] = React.useState({});
      const [inputMessage, setInputMessage] = React.useState("");
      const fileInputRef = React.useRef(null);

      const addCrop = (crop) => {
        if (!selectedCrops.includes(crop)) setSelectedCrops([...selectedCrops, crop]);
      };

      const removeCrop = (crop) => {
        setSelectedCrops(selectedCrops.filter((c) => c !== crop));
      };

      const addMessage = (moduleId, type, content, data = null) => {
        const newMessage = { id: Date.now(), type, content, data, timestamp: new Date() };
        setMessages((prev) => ({ ...prev, [moduleId]: [...(prev[moduleId] || []), newMessage] }));
      };

      const handleSendMessage = (moduleId, message) => {
        if (!message.trim()) return;
        addMessage(moduleId, "user", message);
        setIsProcessing((prev) => ({ ...prev, [moduleId]: true }));
        
        setTimeout(() => {
          const cropContext = selectedCrops.length > 0 ? ` for ${selectedCrops.join(", ")}` : "";
          let response = `For${cropContext}, here's expert advice: ${message}. Ensure proper irrigation, balanced NPK fertilizer, and regular pest monitoring.`;
          
          addMessage(moduleId, "bot", response);
          setIsProcessing((prev) => ({ ...prev, [moduleId]: false }));
        }, 2000);
      };

      const handleUploadImage = (moduleId, file) => {
        setIsProcessing((prev) => ({ ...prev, [moduleId]: true }));
        const reader = new FileReader();
        reader.onload = () => {
          addMessage(moduleId, "user", "Analyzing image...");
          setTimeout(() => {
            const result = {
              caption: "Healthy rice field with proper irrigation",
              health: "Good",
              soil_health: "Optimal",
              diseases: [{ name: "Rice Blast", confidence: 92, severity: "High", symptoms: "Diamond-shaped lesions", treatment: "Apply Tricyclazole @ 0.6g/L" }],
              recommendations: ["Maintain water level (2-5cm)", "Apply NPK fertilizer", "Monitor for pests"]
            };
            addMessage(moduleId, "bot", "Analysis Complete", result);
            setIsProcessing((prev) => ({ ...prev, [moduleId]: false }));
          }, 2000);
        };
        reader.readAsDataURL(file);
      };

      const activeModuleData = activeModule ? aiModules.find((m) => m.id === activeModule) : null;

      return (
        <div className="min-h-screen flex">
          <aside className={`${isSidebarOpen ? "w-80" : "w-0"} transition-all duration-300 bg-black/20 backdrop-blur-xl border-r border-white/20 overflow-hidden`}>
            <div className="p-6 space-y-6 h-screen flex flex-col">
              <div className="flex items-center space-x-3">
                <div className="w-10 h-10 rounded-xl bg-gradient-to-br from-emerald-500 to-green-500 flex items-center justify-center">
                  <Leaf className="h-6 w-6 text-white" />
                </div>
                <div>
                  <h1 className="text-xl font-bold text-white">Smart Farming</h1>
                  <p className="text-xs text-white/70">AI Assistant</p>
                </div>
              </div>

              <div>
                <label className="text-sm text-white/80 mb-2 block">Language</label>
                <select
                  value={selectedLanguage}
                  onChange={(e) => setSelectedLanguage(e.target.value)}
                  className="w-full px-3 py-2 bg-white/10 border border-white/20 rounded-lg text-white"
                >
                  {Object.entries(languages).map(([key, lang]) => (
                    <option key={key} value={key} className="bg-gray-900">{lang.name}</option>
                  ))}
                </select>
              </div>

              <div className="flex-1 overflow-hidden">
                <label className="text-sm text-white/80 mb-2 block">Select Crops</label>
                {selectedCrops.length > 0 && (
                  <div className="flex flex-wrap gap-2 mb-3">
                    {selectedCrops.map((crop) => (
                      <span key={crop} className="px-2 py-1 bg-emerald-500/30 rounded-full text-xs text-white flex items-center gap-1">
                        {cropDatabase[crop].names[languages[selectedLanguage].code]}
                        <X className="h-3 w-3 cursor-pointer" onClick={() => removeCrop(crop)} />
                      </span>
                    ))}
                  </div>
                )}
                <div className="space-y-2 overflow-y-auto scrollbar-hide max-h-64">
                  {Object.entries(cropDatabase).filter(([name]) => !selectedCrops.includes(name)).map(([name, crop]) => (
                    <button
                      key={name}
                      onClick={() => addCrop(name)}
                      className="w-full text-left p-3 bg-white/5 border border-white/20 rounded-lg hover:bg-white/10 transition-all"
                    >
                      <div className="text-white font-medium">{crop.names[languages[selectedLanguage].code]}</div>
                      <div className="text-xs text-white/60">{crop.season} • {crop.category}</div>
                    </button>
                  ))}
                </div>
              </div>
            </div>
          </aside>

          <main className="flex-1 flex flex-col">
            <header className="p-4 bg-black/20 backdrop-blur-xl border-b border-white/20 flex items-center justify-between">
              <button onClick={() => setIsSidebarOpen(!isSidebarOpen)} className="p-2 text-white hover:bg-white/10 rounded-lg">
                <Menu className="h-5 w-5" />
              </button>
              <h2 className="text-2xl font-bold text-white">{activeModuleData ? activeModuleData.name : "AI Farming Modules"}</h2>
              <div className="w-10" />
            </header>

            <div className="flex-1 overflow-auto p-6">
              {!activeModule ? (
                <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 max-w-7xl mx-auto">
                  {aiModules.map((module) => (
                    <div
                      key={module.id}
                      onClick={() => setActiveModule(module.id)}
                      className={`bg-gradient-to-br ${module.colorScheme.bg} backdrop-blur-xl border ${module.colorScheme.border} rounded-2xl p-6 cursor-pointer hover:scale-105 transition-all`}
                    >
                      <div className={`w-16 h-16 rounded-2xl bg-gradient-to-br ${module.colorScheme.gradient} flex items-center justify-center mb-4`}>
                        <module.icon className="h-8 w-8 text-white" />
                      </div>
                      <h3 className="text-lg font-bold text-white mb-2">{module.name}</h3>
                      <p className="text-sm text-white/70 mb-3">{module.description}</p>
                      <span className="text-xs text-white/50">{module.model}</span>
                    </div>
                  ))}
                </div>
              ) : activeModuleData && (
                <div className="h-full max-w-6xl mx-auto flex flex-col">
                  <button onClick={() => setActiveModule(null)} className="mb-4 px-4 py-2 text-white hover:bg-white/10 rounded-lg w-fit">
                    ← Back to Modules
                  </button>
                  <div className="flex-1 bg-white/10 backdrop-blur-xl border border-white/20 rounded-2xl overflow-hidden flex flex-col">
                    <div className="flex-1 overflow-y-auto p-6 space-y-4 scrollbar-hide">
                      {(!messages[activeModuleData.id] || messages[activeModuleData.id].length === 0) && (
                        <div className="text-center py-12">
                          <div className={`w-20 h-20 mx-auto mb-4 rounded-2xl bg-gradient-to-br ${activeModuleData.colorScheme.gradient} flex items-center justify-center`}>
                            <activeModuleData.icon className="h-10 w-10 text-white" />
                          </div>
                          <h3 className="text-xl font-bold text-white mb-2">{activeModuleData.name}</h3>
                          <p className="text-white/70">{activeModuleData.description}</p>
                        </div>
                      )}

                      {(messages[activeModuleData.id] || []).map((msg) => (
                        <div key={msg.id} className={`flex ${msg.type === "user" ? "justify-end" : "justify-start"}`}>
                          <div className={`flex items-start space-x-3 max-w-3xl ${msg.type === "user" ? "flex-row-reverse space-x-reverse" : ""}`}>
                            <div className={`w-10 h-10 rounded-full flex items-center justify-center ${msg.type === "user" ? "bg-blue-500" : `bg-gradient-to-br ${activeModuleData.colorScheme.gradient}`}`}>
                              {msg.type === "user" ? <User className="h-5 w-5 text-white" /> : <Bot className="h-5 w-5 text-white" />}
                            </div>
                            <div className={`rounded-2xl p-4 ${msg.type === "user" ? "bg-blue-500/30 border border-blue-400/40" : `${activeModuleData.colorScheme.bg} border ${activeModuleData.colorScheme.border}`}`}>
                              <div className="text-white">{msg.content}</div>
                              {msg.data && (
                                <div className="mt-3 space-y-2">
                                  <div className="text-sm text-white/90">{msg.data.caption}</div>
                                  {msg.data.diseases?.map((d, i) => (
                                    <div key={i} className="bg-red-500/20 p-3 rounded-lg text-sm">
                                      <div className="font-semibold text-white">{d.name} ({d.confidence}%)</div>
                                      <div className="text-white/80">{d.symptoms}</div>
                                      <div className="text-emerald-300 mt-1">Treatment: {d.treatment}</div>
                                    </div>
                                  ))}
                                  {msg.data.recommendations && (
                                    <div className="bg-green-500/20 p-3 rounded-lg">
                                      <div className="font-semibold text-white mb-1">Recommendations:</div>
                                      {msg.data.recommendations.map((r, i) => (
                                        <div key={i} className="text-sm text-white/80">• {r}</div>
                                      ))}
                                    </div>
                                  )}
                                </div>
                              )}
                              <div className="text-xs text-white/50 mt-2">{msg.timestamp.toLocaleTimeString()}</div>
                            </div>
                          </div>
                        </div>
                      ))}

                      {isProcessing[activeModuleData.id] && (
                        <div className="flex justify-start">
                          <div className="bg-white/10 rounded-2xl p-4 border border-white/20">
                            <div className="flex items-center space-x-2 text-white">
                              <div className="animate-spin h-4 w-4 border-2 border-white border-t-transparent rounded-full"></div>
                              <span>Processing...</span>
                            </div>
                          </div>
                        </div>
                      )}
                    </div>

                    <div className="p-4 bg-white/5 border-t border-white/10">
                      {(activeModuleData.id === "image-analysis" || activeModuleData.id === "visual-search") && (
                        <div className="mb-3">
                          <button
                            onClick={() => fileInputRef.current?.click()}
                            className={`w-full py-3 bg-gradient-to-r ${activeModuleData.colorScheme.gradient} text-white rounded-xl flex items-center justify-center gap-2 hover:opacity-90`}
                          >
                            <Upload className="h-5 w-5" />
                            Upload Image
                          </button>
                          <input
                            ref={fileInputRef}
                            type="file"
                            accept="image/*"
                            onChange={(e) => handleUploadImage(activeModuleData.id, e.target.files[0])}
                            className="hidden"
                          />
                        </div>
                      )}
                      <div className="flex gap-2">
                        <input
                          value={inputMessage}
                          onChange={(e) => setInputMessage(e.target.value)}
                          onKeyPress={(e) => e.key === "Enter" && handleSendMessage(activeModuleData.id, inputMessage) && setInputMessage("")}
                          placeholder="Type your message..."
                          className="flex-1 px-4 py-2 bg-white/10 border border-white/20 rounded-lg text-white placeholder:text-white/60"
                        />
                        <button
                          onClick={() => { handleSendMessage(activeModuleData.id, inputMessage); setInputMessage(""); }}
                          className={`px-6 py-2 bg-gradient-to-r ${activeModuleData.colorScheme.gradient} rounded-lg`}
                        >
                          <Send className="h-5 w-5 text-white" />
                        </button>
                      </div>
                    </div>
                  </div>
                </div>
              )}
            </div>
          </main>
        </div>
      );
    }

    ReactDOM.render(<SmartFarmingApp />, document.getElementById("root"));
  </script>
</body>
</html>
