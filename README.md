// SmartFarmingApp.tsx
// Smart Farming AI Assistant - Complete Single File Implementation
// Features: 6 AI Modules, Multilingual Support (6 Indian Languages), Glassmorphic Design
import React, { useState, useRef, useEffect } from "react";
import { Camera, Sparkles, Video, Music, Globe, Search, Leaf, Menu, Send, Upload, User, Bot, X, Plus } from "lucide-react";
/* ============================================================
   TYPE DEFINITIONS & SHARED SCHEMA
   ============================================================ */
export const languages = {
  english: { name: "English", code: "en" },
  tamil: { name: "தமிழ்", code: "ta" },
  hindi: { name: "हिंदी", code: "hi" },
  telugu: { name: "తెలుగు", code: "te" },
  kannada: { name: "ಕನ್ನಡ", code: "kn" },
  malayalam: { name: "മലയാളം", code: "ml" },
} as const;
export type Language = keyof typeof languages;
export const cropDatabase = {
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
} as const;
export type CropName = keyof typeof cropDatabase;
export interface Message {
  id: number;
  type: "user" | "bot";
  content: string;
  data?: any;
  timestamp: Date;
}
export interface ModuleColors {
  gradient: string;
  bg: string;
  border: string;
  text: string;
}
/* ============================================================
   LANGUAGE SELECTOR COMPONENT
   ============================================================ */
interface LanguageSelectorProps {
  selectedLanguage: Language;
  onLanguageChange: (language: Language) => void;
}
function LanguageSelector({ selectedLanguage, onLanguageChange }: LanguageSelectorProps) {
  const [isOpen, setIsOpen] = useState(false);
  return (
    <div className="relative">
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="w-full px-4 py-2 bg-white/10 backdrop-blur-sm border border-white/20 rounded-lg text-white flex items-center justify-between hover:bg-white/20 transition-all"
      >
        <div className="flex items-center space-x-2">
          <Globe className="h-4 w-4" />
          <span>{languages[selectedLanguage].name}</span>
        </div>
        <span className="text-white/60">▼</span>
      </button>
      {isOpen && (
        <div className="absolute top-full left-0 right-0 mt-2 bg-black/80 backdrop-blur-xl border border-white/20 rounded-lg overflow-hidden z-50">
          {Object.entries(languages).map(([key, lang]) => (
            <button
              key={key}
              onClick={() => {
                onLanguageChange(key as Language);
                setIsOpen(false);
              }}
              className="w-full px-4 py-2 text-left text-white hover:bg-white/10 transition-all"
            >
              {lang.name}
            </button>
          ))}
        </div>
      )}
    </div>
  );
}
/* ============================================================
   CROP SELECTOR COMPONENT
   ============================================================ */
interface CropSelectorProps {
  selectedCrops: CropName[];
  selectedLanguage: Language;
  onAddCrop: (crop: CropName) => void;
  onRemoveCrop: (crop: CropName) => void;
}
function CropSelector({ selectedCrops, selectedLanguage, onAddCrop, onRemoveCrop }: CropSelectorProps) {
  const [searchTerm, setSearchTerm] = useState("");
  const filteredCrops = (Object.keys(cropDatabase) as CropName[]).filter((crop) => {
    const cropData = cropDatabase[crop];
    const currentLang = languages[selectedLanguage].code;
    return (
      cropData.names[currentLang].toLowerCase().includes(searchTerm.toLowerCase()) ||
      crop.toLowerCase().includes(searchTerm.toLowerCase())
    );
  });
  return (
    <div className="bg-white/10 backdrop-blur-sm border border-white/20 rounded-xl p-4 h-full flex flex-col">
      <h3 className="text-lg font-bold text-white mb-3">Select Your Crops</h3>
      <div className="relative mb-3">
        <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 h-4 w-4 text-white/60" />
        <input
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
          placeholder="Search crops..."
          className="w-full pl-10 pr-4 py-2 bg-white/10 border border-white/20 rounded-lg text-white placeholder:text-white/60"
        />
      </div>
      {selectedCrops.length > 0 && (
        <div className="mb-3">
          <p className="text-sm text-white/80 mb-2">Selected Crops</p>
          <div className="flex flex-wrap gap-2">
            {selectedCrops.map((crop) => (
              <span
                key={crop}
                className="px-3 py-1 bg-gradient-to-r from-indigo-500 to-blue-500 text-white rounded-full text-sm flex items-center gap-1"
              >
                {cropDatabase[crop].names[languages[selectedLanguage].code]}
                <button onClick={() => onRemoveCrop(crop)} className="hover:bg-white/20 rounded-full p-0.5">
                  <X className="h-3 w-3" />
                </button>
              </span>
            ))}
          </div>
        </div>
      )}
      <div className="flex-1 overflow-y-auto space-y-2 custom-scrollbar">
        {filteredCrops
          .filter((crop) => !selectedCrops.includes(crop))
          .map((crop) => {
            const cropData = cropDatabase[crop];
            return (
              <button
                key={crop}
                onClick={() => {
                  onAddCrop(crop);
                  setSearchTerm("");
                }}
                className="w-full text-left p-3 bg-white/5 border border-white/20 rounded-lg hover:bg-white/10 transition-all"
              >
                <div className="flex items-center justify-between">
                  <div>
                    <h4 className="font-semibold text-white">{cropData.names[languages[selectedLanguage].code]}</h4>
                    <p className="text-sm text-white/70">
                      {cropData.season} • {cropData.category}
                    </p>
                  </div>
                  <Plus className="h-5 w-5 text-white/60" />
                </div>
              </button>
            );
          })}
      </div>
    </div>
  );
}
/* ============================================================
   MODULE CARD COMPONENT
   ============================================================ */
interface ModuleCardProps {
  id: string;
  name: string;
  icon: any;
  description: string;
  colorScheme: ModuleColors;
  isActive: boolean;
  onClick: () => void;
}
function ModuleCard({ id, name, icon: Icon, description, colorScheme, isActive, onClick }: ModuleCardProps) {
  return (
    <div
      onClick={onClick}
      className={`bg-white/10 backdrop-blur-sm border cursor-pointer transition-all hover:bg-white/20 rounded-2xl p-6 ${
        isActive ? `border-2 ${colorScheme.border} shadow-lg` : "border-white/20"
      }`}
    >
      <div className={`w-16 h-16 rounded-2xl bg-gradient-to-br ${colorScheme.gradient} flex items-center justify-center mb-4 shadow-lg`}>
        <Icon className="h-8 w-8 text-white" />
      </div>
      <h3 className="text-lg font-bold text-white mb-2">{name}</h3>
      <p className="text-sm text-white/70">{description}</p>
    </div>
  );
}
/* ============================================================
   MESSAGE BUBBLE COMPONENT
   ============================================================ */
interface MessageBubbleProps {
  message: Message;
  colorScheme: ModuleColors;
  moduleId: string;
}
function MessageBubble({ message, colorScheme, moduleId }: MessageBubbleProps) {
  return (
    <div className={`flex ${message.type === "user" ? "justify-end" : "justify-start"} mb-4`}>
      <div className={`flex items-start space-x-3 max-w-3xl ${message.type === "user" ? "flex-row-reverse space-x-reverse" : ""}`}>
        <div
          className={`flex-shrink-0 w-10 h-10 rounded-full flex items-center justify-center ${
            message.type === "user" ? "bg-blue-500 shadow-lg" : `bg-gradient-to-br ${colorScheme.gradient} shadow-lg`
          }`}
        >
          {message.type === "user" ? <User className="h-5 w-5 text-white" /> : <Bot className="h-5 w-5 text-white" />}
        </div>
        <div
          className={`rounded-2xl p-4 backdrop-blur-sm ${
            message.type === "user" ? "bg-blue-500/30 border border-blue-400/40 shadow-lg" : `${colorScheme.bg} border ${colorScheme.border} shadow-lg`
          }`}
        >
          <div className="text-white whitespace-pre-line">{message.content}</div>
          {message.data && moduleId === "image-analysis" && (
            <div className="mt-4 space-y-3">
              <div className="bg-white/10 rounded-xl p-4 border border-white/20 backdrop-blur-sm">
                <h4 className="font-bold text-white mb-2">Analysis Results</h4>
                <p className="text-sm text-gray-200 mb-3">{message.data.caption}</p>
                <div className="grid grid-cols-2 gap-3 text-center">
                  <div className="bg-white/10 rounded-lg p-2 backdrop-blur-sm">
                    <div className="text-xl font-bold text-green-300">{message.data.health}</div>
                    <div className="text-xs text-gray-300">Crop Health</div>
                  </div>
                  <div className="bg-white/10 rounded-lg p-2 backdrop-blur-sm">
                    <div className="text-xl font-bold text-blue-300">{message.data.soil_health}</div>
                    <div className="text-xs text-gray-300">Soil Health</div>
                  </div>
                </div>
              </div>
              {message.data.diseases?.map((disease: any, idx: number) => (
                <div key={idx} className="bg-red-500/20 rounded-xl p-4 border border-red-400/30 backdrop-blur-sm">
                  <div className="flex justify-between items-start mb-2">
                    <h5 className="font-semibold text-red-200">{disease.name}</h5>
                    <span className="px-2 py-1 bg-red-500/30 text-red-100 rounded-full text-xs">{disease.severity}</span>
                  </div>
                  <p className="text-sm text-gray-200">
                    <strong>Confidence:</strong> {disease.confidence}%
                  </p>
                  <p className="text-sm text-gray-200">
                    <strong>Symptoms:</strong> {disease.symptoms}
                  </p>
                  <p className="text-sm text-gray-200">
                    <strong>Treatment:</strong> {disease.treatment}
                  </p>
                </div>
              ))}
              {message.data.recommendations && (
                <div className="bg-green-500/20 rounded-xl p-4 border border-green-400/30 backdrop-blur-sm">
                  <h5 className="font-semibold text-green-200 mb-2">Recommendations</h5>
                  <ul className="space-y-1">
                    {message.data.recommendations.map((rec: string, idx: number) => (
                      <li key={idx} className="text-sm text-gray-200">
                        • {rec}
                      </li>
                    ))}
                  </ul>
                </div>
              )}
            </div>
          )}
          <div className="text-xs text-gray-300 mt-2 text-right">{message.timestamp.toLocaleTimeString()}</div>
        </div>
      </div>
    </div>
  );
}
/* ============================================================
   CHAT INTERFACE COMPONENT
   ============================================================ */
interface ChatInterfaceProps {
  moduleId: string;
  moduleName: string;
  icon: any;
  model: string;
  description: string;
  colorScheme: ModuleColors;
  messages: Message[];
  isProcessing: boolean;
  capturedImage?: string;
  onSendMessage: (message: string) => void;
  onUploadImage?: (file: File) => void;
  showImageUpload?: boolean;
}
function ChatInterface({
  moduleId,
  moduleName,
  icon: Icon,
  model,
  description,
  colorScheme,
  messages,
  isProcessing,
  capturedImage,
  onSendMessage,
  onUploadImage,
  showImageUpload = false,
}: ChatInterfaceProps) {
  const [inputMessage, setInputMessage] = useState("");
  const fileInputRef = useRef<HTMLInputElement>(null);
  const messagesEndRef = useRef<HTMLDivElement>(null);
  useEffect(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [messages, isProcessing]);
  const handleSend = () => {
    if (inputMessage.trim()) {
      onSendMessage(inputMessage);
      setInputMessage("");
    }
  };
  const handleFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (file && onUploadImage) {
      onUploadImage(file);
    }
  };
  return (
    <div className="flex flex-col h-full">
      <div className="flex-1 overflow-y-auto p-6 custom-scrollbar">
        {messages.length === 0 && (
          <div className="text-center py-12">
            <div className={`w-20 h-20 mx-auto mb-4 rounded-2xl bg-gradient-to-br ${colorScheme.gradient} flex items-center justify-center shadow-lg`}>
              <Icon className="h-10 w-10 text-white" />
            </div>
            <h3 className="text-xl font-bold text-white mb-2">{moduleName}</h3>
            <p className="text-gray-200">{description}</p>
            <div className="mt-4 px-4 py-2 bg-white/10 rounded-lg inline-block backdrop-blur-sm">
              <p className="text-sm text-white">Model: {model}</p>
            </div>
          </div>
        )}
        {messages.map((msg) => (
          <MessageBubble key={msg.id} message={msg} colorScheme={colorScheme} moduleId={moduleId} />
        ))}
        {isProcessing && (
          <div className="flex justify-start mb-4">
            <div className="flex items-center space-x-3">
              <div className={`w-10 h-10 rounded-full flex items-center justify-center bg-gradient-to-br ${colorScheme.gradient} shadow-lg`}>
                <Bot className="h-5 w-5 text-white" />
              </div>
              <div className="bg-white/10 rounded-2xl p-4 border border-white/20 backdrop-blur-sm shadow-lg">
                <div className="flex items-center space-x-3">
                  <div className="animate-spin rounded-full h-5 w-5 border-2 border-white border-t-transparent"></div>
                  <span className="text-white">Processing with AI...</span>
                </div>
              </div>
            </div>
          </div>
        )}
        <div ref={messagesEndRef} />
      </div>
      {capturedImage && (
        <div className="p-4 bg-white/5 border-t border-white/10">
          <img src={capturedImage} alt="Uploaded" className="max-w-xs rounded-lg border-2 border-white/30 shadow-lg" />
        </div>
      )}
      <div className="p-6 bg-white/5 border-t border-white/10 backdrop-blur-sm">
        {showImageUpload && (
          <div className="mb-3">
            <button
              onClick={() => fileInputRef.current?.click()}
              disabled={isProcessing}
              className={`w-full px-4 py-3 bg-gradient-to-r ${colorScheme.gradient} text-white rounded-xl hover:opacity-90 disabled:opacity-50 transition-all shadow-lg flex items-center justify-center space-x-2`}
            >
              <Upload className="h-5 w-5" />
              <span>{isProcessing ? "Processing..." : "Upload Field Image"}</span>
            </button>
            <input ref={fileInputRef} type="file" accept="image/*" onChange={handleFileChange} className="hidden" />
          </div>
        )}
        <div className="flex space-x-3">
          <input
            value={inputMessage}
            onChange={(e) => setInputMessage(e.target.value)}
            onKeyPress={(e) => e.key === "Enter" && handleSend()}
            placeholder="Type your message..."
            disabled={isProcessing}
            className="flex-1 px-4 py-2 bg-white/10 border border-white/20 rounded-lg text-white placeholder:text-white/60"
          />
          <button
            onClick={handleSend}
            disabled={isProcessing || !inputMessage.trim()}
            className={`px-6 py-2 bg-gradient-to-r ${colorScheme.gradient} text-white rounded-lg hover:opacity-90 disabled:opacity-50 transition-all shadow-lg`}
          >
            <Send className="h-5 w-5" />
          </button>
        </div>
      </div>
    </div>
  );
}
/* ============================================================
   MAIN APP COMPONENT
   ============================================================ */
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
export default function SmartFarmingApp() {
  const [selectedLanguage, setSelectedLanguage] = useState<Language>("english");
  const [selectedCrops, setSelectedCrops] = useState<CropName[]>([]);
  const [activeModule, setActiveModule] = useState<string | null>(null);
  const [isSidebarOpen, setIsSidebarOpen] = useState(true);
  const [messages, setMessages] = useState<Record<string, Message[]>>({});
  const [isProcessing, setIsProcessing] = useState<Record<string, boolean>>({});
  const [capturedImages, setCapturedImages] = useState<Record<string, string>>({});
  const addCrop = (crop: CropName) => {
    if (!selectedCrops.includes(crop)) setSelectedCrops([...selectedCrops, crop]);
  };
  const removeCrop = (crop: CropName) => {
    setSelectedCrops(selectedCrops.filter((c) => c !== crop));
  };
  const addMessage = (moduleId: string, type: "user" | "bot", content: string, data?: any) => {
    const newMessage: Message = {
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
  const handleSendMessage = (moduleId: string, message: string) => {
    addMessage(moduleId, "user", message);
    setIsProcessing((prev) => ({ ...prev, [moduleId]: true }));
    setTimeout(() => {
      let response = "";
      if (moduleId === "crop-advisor") {
        const cropContext = selectedCrops.length > 0 ? ` for ${selectedCrops.join(", ")}` : "";
        response = `For${cropContext}, here's expert advice: ${message}. Ensure proper irrigation, use balanced NPK fertilizer (10:26:26), and monitor for pests regularly. Organic manure is recommended at 10-15 tons/hectare.`;
      } else if (moduleId === "text-translator") {
        response = `Translation: This is a sample translation in your selected language. In production, real AI translation would occur here.`;
      } else if (moduleId === "voice-assistant") {
        response = "Audio advice generated. In production, you would hear farming advice in your selected language.";
      } else if (moduleId === "video-guide") {
        response = "Video generation initiated. This process may take a few minutes. In production, a farming technique video would be generated.";
      } else {
        response = "Thank you for your query. In production, AI would provide detailed farming assistance.";
      }
      addMessage(moduleId, "bot", response);
      setIsProcessing((prev) => ({ ...prev, [moduleId]: false }));
    }, 2000);
  };
  const handleUploadImage = (moduleId: string, file: File) => {
    setIsProcessing((prev) => ({ ...prev, [moduleId]: true }));
    const reader = new FileReader();
    reader.onload = (e) => {
      const imageDataUrl = e.target?.result as string;
      setCapturedImages((prev) => ({ ...prev, [moduleId]: imageDataUrl }));
      addMessage(moduleId, "user", "Analyzing field image...");
      setTimeout(() => {
        const analysisResult = {
          caption: "Healthy rice field with proper irrigation",
          health: "Good",
          soil_health: "Optimal",
          diseases: [
            {
              name: "Rice Blast",
              confidence: 92,
              severity: "High",
              symptoms: "Diamond-shaped lesions on leaves",
              treatment: "Apply Tricyclazole @ 0.6g/L",
            },
          ],
          recommendations: [
            "Maintain proper water level (2-5cm)",
            "Apply balanced NPK fertilizer",
            "Monitor for leaf blast symptoms",
            "Ensure proper spacing between plants",
          ],
        };
        addMessage(moduleId, "bot", "Image Analysis Complete", analysisResult);
        setIsProcessing((prev) => ({ ...prev, [moduleId]: false }));
      }, 3000);
    };
    reader.readAsDataURL(file);
  };
  const activeModuleData = activeModule ? aiModules.find((m) => m.id === activeModule) : null;
  return (
    <div className="min-h-screen relative overflow-hidden">
      {/* Animated Gradient Background */}
      <style>{`
        @keyframes gradient {
          0% { background-position: 0% 50%; }
          50% { background-position: 100% 50%; }
          100% { background-position: 0% 50%; }
        }
        .custom-scrollbar::-webkit-scrollbar {
          width: 6px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
          background: rgba(255, 255, 255, 0.1);
          border-radius: 10px;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
          background: rgba(255, 255, 255, 0.3);
          border-radius: 10px;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover {
          background: rgba(255, 255, 255, 0.5);
        }
      `}</style>
      <div
        className="absolute inset-0 z-0"
        style={{
          backgroundImage: `linear-gradient(-45deg, #ee7752, #e73c7e, #23a6d5, #23d5ab)`,
          backgroundSize: "400% 400%",
          animation: "gradient 15s ease infinite",
        }}
      />
      <div className="relative z-10 flex h-screen">
        {/* Sidebar */}
        <aside
          className={`${
            isSidebarOpen ? "w-80" : "w-0"
          } transition-all duration-300 overflow-hidden bg-black/20 backdrop-blur-xl border-r border-white/20`}
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
              <CropSelector selectedCrops={selectedCrops} selectedLanguage={selectedLanguage} onAddCrop={addCrop} onRemoveCrop={removeCrop} />
            </div>
          </div>
        </aside>
        {/* Main Content */}
        <main className="flex-1 flex flex-col overflow-hidden">
          <header className="p-4 bg-black/20 backdrop-blur-xl border-b border-white/20 flex items-center justify-between">
            <button
              onClick={() => setIsSidebarOpen(!isSidebarOpen)}
              className="p-2 text-white hover:bg-white/10 rounded-lg transition-all"
            >
              <Menu className="h-5 w-5" />
            </button>
            <h2 className="text-2xl font-bold text-white">{activeModuleData ? activeModuleData.name : "AI Farming Modules"}</h2>
            <div className="w-10" />
          </header>
          <div className="flex-1 overflow-auto p-6 custom-scrollbar">
            {!activeModule ? (
              <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 max-w-7xl mx-auto">
                {aiModules.map((module) => (
                  <ModuleCard key={module.id} {...module} isActive={false} onClick={() => setActiveModule(module.id)} />
                ))}
              </div>
            ) : (
              activeModuleData && (
                <div className="h-full max-w-6xl mx-auto">
                  <button onClick={() => setActiveModule(null)} className="mb-4 px-4 py-2 text-white hover:bg-white/10 rounded-lg transition-all">
                    ← Back to Modules
                  </button>
                  <div className="bg-white/10 backdrop-blur-sm border border-white/20 rounded-2xl h-[calc(100%-4rem)] overflow-hidden">
                    <ChatInterface
                      moduleId={activeModuleData.id}
                      moduleName={activeModuleData.name}
                      icon={activeModuleData.icon}
                      model={activeModuleData.model}
                      description={activeModuleData.description}
                      colorScheme={activeModuleData.colorScheme}
                      messages={messages[activeModuleData.id] || []}
                      isProcessing={isProcessing[activeModuleData.id] || false}
                      capturedImage={capturedImages[activeModuleData.id]}
                      onSendMessage={(msg) => handleSendMessage(activeModuleData.id, msg)}
                      onUploadImage={
                        activeModuleData.id === "image-analysis" || activeModuleData.id === "visual-search"
                          ? (file) => handleUploadImage(activeModuleData.id, file)
                          : undefined
                      }
                      showImageUpload={activeModuleData.id === "image-analysis" || activeModuleData.id === "visual-search"}
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
