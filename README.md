import React, { useState } from 'react';
import { Camera, Sparkles, Video, Music, Globe, Search, Leaf, Menu, X, Upload, Send, Loader } from 'lucide-react';

// Shared Schema
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
    water: "High",
    soil: "Clay loam",
  },
  Wheat: {
    names: { en: "Wheat", hi: "गेहूं", ta: "கோதுமை", te: "గోధుమ", kn: "ಗೋಧಿ", ml: "ഗോതമ്പ്" },
    season: "Rabi",
    compatible: ["Mustard", "Gram"],
    incompatible: ["Rice"],
    category: "cereal",
    water: "Medium",
    soil: "Well-drained loam",
  },
  Maize: {
    names: { en: "Maize", hi: "मक्का", ta: "மக்காச்சோளம்", te: "మొక్కజొన్న", kn: "ಜೋಳ", ml: "ചോളം" },
    season: "Kharif/Rabi",
    compatible: ["Beans", "Pumpkin"],
    incompatible: [],
    category: "cereal",
    water: "Medium",
    soil: "Loamy soil",
  },
  Tomato: {
    names: { en: "Tomato", hi: "टमाटर", ta: "தக்காளி", te: "టమాటో", kn: "ಟೊಮೇಟೊ", ml: "തക്കാളി" },
    season: "Rabi/Summer",
    compatible: ["Basil", "Marigold"],
    incompatible: ["Potato"],
    category: "vegetable",
    water: "Regular",
    soil: "Sandy loam",
  },
  Cotton: {
    names: { en: "Cotton", hi: "कपास", ta: "பருத்தி", te: "పత్తి", kn: "ಹತ್ತಿ", ml: "പരുത്തി" },
    season: "Kharif",
    compatible: ["Marigold", "Chili"],
    incompatible: [],
    category: "fiber_crop",
    water: "Medium",
    soil: "Black soil",
  },
  Sugarcane: {
    names: { en: "Sugarcane", hi: "गन्ना", ta: "கரும்பு", te: "చెరకు", kn: "ಕಬ್ಬು", ml: "കരിമ്പു" },
    season: "Annual",
    compatible: ["Potato", "Onion"],
    incompatible: ["Wheat"],
    category: "cash_crop",
    water: "High",
    soil: "Deep loamy soil",
  },
};

// Language Selector Component
const LanguageSelector = ({ selectedLanguage, onLanguageChange }) => (
  <div className="space-y-2">
    <label className="text-sm font-medium text-white/90">Language</label>
    <select
      value={selectedLanguage}
      onChange={(e) => onLanguageChange(e.target.value)}
      className="w-full px-3 py-2 bg-white/10 border border-white/20 rounded-lg text-white focus:outline-none focus:ring-2 focus:ring-emerald-500"
    >
      {Object.entries(languages).map(([key, lang]) => (
        <option key={key} value={key} className="bg-gray-900">
          {lang.name}
        </option>
      ))}
    </select>
  </div>
);

// Crop Selector Component
const CropSelector = ({ selectedCrops, selectedLanguage, onAddCrop, onRemoveCrop }) => {
  const [searchTerm, setSearchTerm] = useState('');
  const langCode = languages[selectedLanguage].code;

  const filteredCrops = Object.entries(cropDatabase).filter(([name, crop]) =>
    crop.names[langCode].toLowerCase().includes(searchTerm.toLowerCase())
  );

  return (
    <div className="space-y-3">
      <label className="text-sm font-medium text-white/90">Your Crops</label>
      <input
        type="text"
        placeholder="Search crops..."
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        className="w-full px-3 py-2 bg-white/10 border border-white/20 rounded-lg text-white placeholder-white/50 focus:outline-none focus:ring-2 focus:ring-emerald-500"
      />
      <div className="flex flex-wrap gap-2">
        {selectedCrops.map((crop) => (
          <span
            key={crop}
            className="px-3 py-1 bg-emerald-500/30 border border-emerald-400/40 rounded-full text-sm text-white flex items-center gap-2"
          >
            {cropDatabase[crop].names[langCode]}
            <X className="h-3 w-3 cursor-pointer" onClick={() => onRemoveCrop(crop)} />
          </span>
        ))}
      </div>
      <div className="max-h-40 overflow-y-auto space-y-1">
        {filteredCrops.map(([name, crop]) => (
          <div
            key={name}
            onClick={() => onAddCrop(name)}
            className="px-3 py-2 bg-white/5 hover:bg-white/10 border border-white/10 rounded-lg cursor-pointer transition-colors text-white text-sm"
          >
            {crop.names[langCode]}
          </div>
        ))}
      </div>
    </div>
  );
};

// Module Card Component
const ModuleCard = ({ id, name, icon: Icon, model, description, colorScheme, onClick }) => (
  <div
    onClick={onClick}
    className={`${colorScheme.bg} backdrop-blur-xl border ${colorScheme.border} rounded-2xl p-6 cursor-pointer transition-all hover:scale-105 hover:shadow-2xl`}
  >
    <div className={`w-12 h-12 rounded-xl bg-gradient-to-br ${colorScheme.gradient} flex items-center justify-center mb-4`}>
      <Icon className="h-6 w-6 text-white" />
    </div>
    <h3 className="text-xl font-bold text-white mb-2">{name}</h3>
    <p className="text-sm text-white/70 mb-3">{description}</p>
    <div className={`inline-flex items-center px-3 py-1 rounded-full ${colorScheme.text} bg-white/10 text-xs font-medium`}>
      {model}
    </div>
  </div>
);

// Chat Interface Component
const ChatInterface = ({
  moduleId,
  moduleName,
  icon: Icon,
  model,
  colorScheme,
  messages,
  isProcessing,
  capturedImage,
  onSendMessage,
  onUploadImage,
  showImageUpload
}) => {
  const [input, setInput] = useState('');
  const fileInputRef = React.useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    if (input.trim()) {
      onSendMessage(input);
      setInput('');
    }
  };

  const handleFileChange = (e) => {
    const file = e.target.files?.[0];
    if (file && onUploadImage) {
      onUploadImage(file);
    }
  };

  return (
    <div className="flex flex-col h-full bg-black/30 backdrop-blur-xl">
      <div className={`p-4 bg-gradient-to-r ${colorScheme.gradient} flex items-center gap-3`}>
        <Icon className="h-6 w-6 text-white" />
        <div>
          <h3 className="text-lg font-bold text-white">{moduleName}</h3>
          <p className="text-xs text-white/80">{model}</p>
        </div>
      </div>

      <div className="flex-1 overflow-y-auto p-4 space-y-4">
        {messages.map((msg) => (
          <div key={msg.id} className={`flex ${msg.type === 'user' ? 'justify-end' : 'justify-start'}`}>
            <div
              className={`max-w-lg px-4 py-3 rounded-2xl ${
                msg.type === 'user'
                  ? 'bg-gradient-to-r ' + colorScheme.gradient + ' text-white'
                  : 'bg-white/10 text-white border border-white/20'
              }`}
            >
              {msg.data ? (
                <div className="space-y-2">
                  <p className="font-semibold">{msg.content}</p>
                  {msg.data.caption && <p className="text-sm opacity-90">{msg.data.caption}</p>}
                  {msg.data.health && (
                    <div className="text-sm">
                      <strong>Health:</strong> {msg.data.health}
                    </div>
                  )}
                  {msg.data.diseases && msg.data.diseases.length > 0 && (
                    <div className="mt-2">
                      <strong className="text-sm">Detected Issues:</strong>
                      {msg.data.diseases.map((disease, idx) => (
                        <div key={idx} className="mt-2 p-2 bg-white/10 rounded-lg text-sm">
                          <div className="font-semibold">{disease.name} ({disease.confidence}%)</div>
                          <div>Severity: {disease.severity}</div>
                          <div className="mt-1">{disease.symptoms}</div>
                          <div className="mt-1 text-emerald-300">Treatment: {disease.treatment}</div>
                        </div>
                      ))}
                    </div>
                  )}
                  {msg.data.recommendations && (
                    <div className="mt-2">
                      <strong className="text-sm">Recommendations:</strong>
                      <ul className="list-disc list-inside text-sm mt-1 space-y-1">
                        {msg.data.recommendations.map((rec, idx) => (
                          <li key={idx}>{rec}</li>
                        ))}
                      </ul>
                    </div>
                  )}
                </div>
              ) : (
                <p>{msg.content}</p>
              )}
            </div>
          </div>
        ))}
        {isProcessing && (
          <div className="flex justify-start">
            <div className="px-4 py-3 bg-white/10 rounded-2xl border border-white/20 flex items-center gap-2 text-white">
              <Loader className="h-4 w-4 animate-spin" />
              <span>Processing...</span>
            </div>
          </div>
        )}
      </div>

      <form onSubmit={handleSubmit} className="p-4 border-t border-white/20">
        <div className="flex gap-2">
          {showImageUpload && (
            <>
              <input
                ref={fileInputRef}
                type="file"
                accept="image/*"
                onChange={handleFileChange}
                className="hidden"
              />
              <button
                type="button"
                onClick={() => fileInputRef.current?.click()}
                className="px-4 py-2 bg-white/10 hover:bg-white/20 border border-white/20 rounded-lg text-white transition-colors"
              >
                <Upload className="h-5 w-5" />
              </button>
            </>
          )}
          <input
            type="text"
            value={input}
            onChange={(e) => setInput(e.target.value)}
            placeholder="Type your message..."
            className="flex-1 px-4 py-2 bg-white/10 border border-white/20 rounded-lg text-white placeholder-white/50 focus:outline-none focus:ring-2 focus:ring-emerald-500"
          />
          <button
            type="submit"
            disabled={isProcessing}
            className={`px-4 py-2 bg-gradient-to-r ${colorScheme.gradient} rounded-lg text-white hover:opacity-90 transition-opacity disabled:opacity-50`}
          >
            <Send className="h-5 w-5" />
          </button>
        </div>
      </form>
    </div>
  );
};

// AI Modules Configuration
const aiModules = [
  {
    id: "image-analysis",
    name: "Field Image Analysis",
    icon: Camera,
    model: "microsoft/resnet-50",
    description: "Analyze field images for disease detection and crop health",
    colorScheme: {
      gradient: "from-pink-500 to-rose-500",
      bg: "bg-gradient-to-br from-pink-500/20 to-rose-500/20",
      border: "border-pink-400/40",
      text: "text-pink-300",
    },
  },
  {
    id: "crop-advisor",
    name: "Intelligent Crop Advisor",
    icon: Sparkles,
    model: "microsoft/DialoGPT-medium",
    description: "Get detailed farming advice and crop recommendations",
    colorScheme: {
      gradient: "from-amber-500 to-orange-500",
      bg: "bg-gradient-to-br from-amber-500/20 to-orange-500/20",
      border: "border-amber-400/40",
      text: "text-amber-300",
    },
  },
  {
    id: "video-guide",
    name: "Video Farming Guides",
    icon: Video,
    model: "Video Generation AI",
    description: "Generate visual farming technique demonstrations",
    colorScheme: {
      gradient: "from-cyan-500 to-sky-500",
      bg: "bg-gradient-to-br from-cyan-500/20 to-sky-500/20",
      border: "border-cyan-400/40",
      text: "text-cyan-300",
    },
  },
  {
    id: "voice-assistant",
    name: "Voice Assistant",
    icon: Music,
    model: "suno/bark",
    description: "Listen to farming advice in your language",
    colorScheme: {
      gradient: "from-emerald-500 to-green-500",
      bg: "bg-gradient-to-br from-emerald-500/20 to-green-500/20",
      border: "border-emerald-400/40",
      text: "text-emerald-300",
    },
  },
  {
    id: "text-translator",
    name: "Multilingual Translator",
    icon: Globe,
    model: "Helsinki-NLP/opus-mt-en-mul",
    description: "Translate farming content to regional languages",
    colorScheme: {
      gradient: "from-violet-500 to-purple-500",
      bg: "bg-gradient-to-br from-violet-500/20 to-purple-500/20",
      border: "border-violet-400/40",
      text: "text-violet-300",
    },
  },
  {
    id: "visual-search",
    name: "Visual Crop Search",
    icon: Search,
    model: "openai/clip-vit-large-patch14",
    description: "Search crops and diseases using images",
    colorScheme: {
      gradient: "from-fuchsia-500 to-pink-500",
      bg: "bg-gradient-to-br from-fuchsia-500/20 to-pink-500/20",
      border: "border-fuchsia-400/40",
      text: "text-fuchsia-300",
    },
  },
];

// Main App Component
export default function SmartFarmingApp() {
  const [selectedLanguage, setSelectedLanguage] = useState('english');
  const [selectedCrops, setSelectedCrops] = useState([]);
  const [activeModule, setActiveModule] = useState(null);
  const [isSidebarOpen, setIsSidebarOpen] = useState(true);
  const [messages, setMessages] = useState({});
  const [isProcessing, setIsProcessing] = useState({});
  const [capturedImages, setCapturedImages] = useState({});

  const addCrop = (crop) => {
    if (!selectedCrops.includes(crop)) setSelectedCrops([...selectedCrops, crop]);
  };

  const removeCrop = (crop) => {
    setSelectedCrops(selectedCrops.filter((c) => c !== crop));
  };

  const addMessage = (moduleId, type, content, data) => {
    const newMessage = {
      id: Date.now(),
      type,
      content,
      data,
      timestamp: new Date(),
    };
    setMessages((prev) => ({
      ...prev,
      [moduleId]: [...(prev[moduleId] || []), newMessage],
    }));
  };

  const handleSendMessage = (moduleId, message) => {
    addMessage(moduleId, 'user', message);
    setIsProcessing((prev) => ({ ...prev, [moduleId]: true }));

    setTimeout(() => {
      let response = '';

      if (moduleId === 'crop-advisor') {
        const cropContext = selectedCrops.length > 0 ? ` for ${selectedCrops.join(', ')}` : '';
        response = `For${cropContext}, here's expert advice: ${message}. Ensure proper irrigation, use balanced NPK fertilizer (10:26:26), and monitor for pests regularly. Organic manure is recommended at 10-15 tons/hectare.`;
      } else if (moduleId === 'text-translator') {
        response = `Translation: This is a sample translation in your selected language.`;
      } else if (moduleId === 'voice-assistant') {
        response = 'Audio advice generated successfully.';
      } else if (moduleId === 'video-guide') {
        response = 'Video generation initiated. Your farming guide will be ready shortly.';
      } else {
        response = 'Thank you for your query. How else can I assist you?';
      }

      addMessage(moduleId, 'bot', response);
      setIsProcessing((prev) => ({ ...prev, [moduleId]: false }));
    }, 2000);
  };

  const handleUploadImage = (moduleId, file) => {
    setIsProcessing((prev) => ({ ...prev, [moduleId]: true }));
    const reader = new FileReader();
    reader.onload = (e) => {
      const imageDataUrl = e.target.result;
      setCapturedImages((prev) => ({ ...prev, [moduleId]: imageDataUrl }));
      addMessage(moduleId, 'user', 'Analyzing field image...');

      setTimeout(() => {
        const analysisResult = {
          caption: 'Healthy rice field with proper irrigation',
          health: 'Good',
          soil_health: 'Optimal',
          diseases: [
            {
              name: 'Rice Blast',
              confidence: 92,
              severity: 'High',
              symptoms: 'Diamond-shaped lesions on leaves',
              treatment: 'Apply Tricyclazole @ 0.6g/L',
            },
          ],
          recommendations: [
            'Maintain proper water level (2-5cm)',
            'Apply balanced NPK fertilizer',
            'Monitor for leaf blast symptoms',
            'Ensure proper spacing between plants',
          ],
        };

        addMessage(moduleId, 'bot', 'Image Analysis Complete', analysisResult);
        setIsProcessing((prev) => ({ ...prev, [moduleId]: false }));
      }, 3000);
    };
    reader.readAsDataURL(file);
  };

  const activeModuleData = activeModule ? aiModules.find((m) => m.id === activeModule) : null;

  return (
    <div className="min-h-screen relative overflow-hidden">
      <style>{`
        @keyframes gradient {
          0% { background-position: 0% 50%; }
          50% { background-position: 100% 50%; }
          100% { background-position: 0% 50%; }
        }
      `}</style>
      
      <div
        className="absolute inset-0 z-0"
        style={{
          backgroundImage: 'linear-gradient(-45deg, #ee7752, #e73c7e, #23a6d5, #23d5ab)',
          backgroundSize: '400% 400%',
          animation: 'gradient 15s ease infinite',
        }}
      />

      <div className="relative z-10 flex h-screen">
        <aside
          className={`${isSidebarOpen ? 'w-80' : 'w-0'} transition-all duration-300 overflow-hidden bg-black/20 backdrop-blur-xl border-r border-white/20`}
        >
          <div className="p-6 space-y-6 h-full flex flex-col">
            <div className="flex items-center space-x-3">
              <div className="w-10 h-10 rounded-xl bg-gradient-to-br from-emerald-500 to-green-500 flex items-center justify-center shadow-lg">
                <Leaf className="h-6 w-6 text-white" />
              </div>
              <div>
                <h1 className="text-xl font-bold text-white">Smart Farming</h1>
                <p className="text-xs text-white/70">AI Assistant</p>
              </div>
            </div>

            <LanguageSelector selectedLanguage={selectedLanguage} onLanguageChange={setSelectedLanguage} />

            <div className="flex-1 overflow-hidden">
              <CropSelector
                selectedCrops={selectedCrops}
                selectedLanguage={selectedLanguage}
                onAddCrop={addCrop}
                onRemoveCrop={removeCrop}
              />
            </div>
          </div>
        </aside>

        <main className="flex-1 flex flex-col overflow-hidden">
          <header className="p-4 bg-black/20 backdrop-blur-xl border-b border-white/20 flex items-center justify-between">
            <button
              onClick={() => setIsSidebarOpen(!isSidebarOpen)}
              className="p-2 hover:bg-white/10 rounded-lg text-white transition-colors"
            >
              <Menu className="h-5 w-5" />
            </button>
            <h2 className="text-2xl font-bold text-white">
              {activeModuleData ? activeModuleData.name : 'AI Farming Modules'}
            </h2>
            <div className="w-10" />
          </header>

          <div className="flex-1 overflow-auto p-6">
            {!activeModule ? (
              <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 max-w-7xl mx-auto">
                {aiModules.map((module) => (
                  <ModuleCard key={module.id} {...module} onClick={() => setActiveModule(module.id)} />
                ))}
              </div>
            ) : (
              activeModuleData && (
                <div className="h-full max-w-6xl mx-auto">
                  <button
                    onClick={() => setActiveModule(null)}
                    className="mb-4 px-4 py-2 hover:bg-white/10 rounded-lg text-white transition-colors"
                  >
                    ← Back to Modules
                  </button>
                  <div className="border border-white/20 rounded-2xl h-[calc(100%-4rem)] overflow-hidden">
                    <ChatInterface
                      moduleId={activeModuleData.id}
                      moduleName={activeModuleData.name}
                      icon={activeModuleData.icon}
                      model={activeModuleData.model}
                      colorScheme={activeModuleData.colorScheme}
                      messages={messages[activeModuleData.id] || []}
                      isProcessing={isProcessing[activeModuleData.id] || false}
                      capturedImage={capturedImages[activeModuleData.id]}
                      onSendMessage={(msg) => handleSendMessage(activeModuleData.id, msg)}
                      onUploadImage={
                        activeModuleData.id === 'image-analysis' || activeModuleData.id === 'visual-search'
                          ? (file) => handleUploadImage(activeModuleData.id, file)
                          : undefined
                      }
                      showImageUpload={
                        activeModuleData.id === 'image-analysis' || activeModuleData.id === 'visual-search'
                      }
                    />
                  </div>
                </div>
              )
            )}
          </div>
        </main>
      </div>
    </div>
  );
}
