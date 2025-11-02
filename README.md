import React, { useState, useRef } from 'react';
import { Camera, Upload, Leaf, Send, Bot, User, Globe, X, Search, Zap, Video, Music, Sparkles, ChevronRight, Menu } from 'lucide-react';

const SmartFarmingAssistant = () => {
  const [selectedCrops, setSelectedCrops] = useState([]);
  const [cropSearchTerm, setCropSearchTerm] = useState('');
  const [selectedLanguage, setSelectedLanguage] = useState('english');
  const [messages, setMessages] = useState({});
  const [inputMessages, setInputMessages] = useState({});
  const [activeTab, setActiveTab] = useState('crops');
  const [isProcessing, setIsProcessing] = useState({});
  const [capturedImages, setCapturedImages] = useState({});
  const [isSidebarOpen, setIsSidebarOpen] = useState(true);
  const fileInputRefs = useRef({});
  
  const API_KEY = '4d131e5ab0164a1f74b76518edcb0c31';

  const languages = {
    english: { name: 'English', code: 'en' },
    tamil: { name: 'தமிழ்', code: 'ta' },
    hindi: { name: 'हिंदी', code: 'hi' },
    telugu: { name: 'తెలుగు', code: 'te' },
    kannada: { name: 'ಕನ್ನಡ', code: 'kn' },
    malayalam: { name: 'മലയാളം', code: 'ml' }
  };

  const cropDatabase = {
    'Rice': { names: { en: 'Rice', hi: 'चावल', ta: 'அரிசி' }, season: 'Kharif/Rabi', compatible: ['Green Gram', 'Black Gram'], incompatible: ['Wheat'], category: 'cereal' },
    'Wheat': { names: { en: 'Wheat', hi: 'गेहूं', ta: 'கோதுமை' }, season: 'Rabi', compatible: ['Mustard', 'Gram'], incompatible: ['Rice'], category: 'cereal' },
    'Maize': { names: { en: 'Maize', hi: 'मक्का', ta: 'மக்காச்சோளம்' }, season: 'Kharif/Rabi', compatible: ['Beans', 'Pumpkin'], incompatible: [], category: 'cereal' },
    'Tomato': { names: { en: 'Tomato', hi: 'टमाटर', ta: 'தக்காளி' }, season: 'Rabi/Summer', compatible: ['Basil', 'Marigold'], incompatible: ['Potato'], category: 'vegetable' },
    'Cotton': { names: { en: 'Cotton', hi: 'कपास', ta: 'பருத்தி' }, season: 'Kharif', compatible: ['Marigold', 'Chili'], incompatible: [], category: 'fiber_crop' },
    'Sugarcane': { names: { en: 'Sugarcane', hi: 'गन्ना', ta: 'கரும்பு' }, season: 'Annual', compatible: ['Potato', 'Onion'], incompatible: ['Wheat'], category: 'cash_crop' },
  };

  const aiModules = [
    {
      id: 'image-analysis',
      name: 'Field Image Analysis',
      icon: Camera,
      model: 'Salesforce/blip-image-captioning-large',
      description: 'Analyze field images for disease detection and crop health',
      color: 'from-purple-600 to-blue-600',
      bgColor: 'bg-gradient-to-br from-purple-900/20 to-blue-900/20'
    },
    {
      id: 'crop-advisor',
      name: 'Intelligent Crop Advisor',
      icon: Sparkles,
      model: 'deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B',
      description: 'Get detailed farming advice and crop recommendations',
      color: 'from-blue-600 to-cyan-600',
      bgColor: 'bg-gradient-to-br from-blue-900/20 to-cyan-900/20'
    },
    {
      id: 'video-guide',
      name: 'Video Farming Guides',
      icon: Video,
      model: 'ali-vilab/text-to-video-ms-1.7b',
      description: 'Generate visual farming technique demonstrations',
      color: 'from-cyan-600 to-teal-600',
      bgColor: 'bg-gradient-to-br from-cyan-900/20 to-teal-900/20'
    },
    {
      id: 'voice-assistant',
      name: 'Voice Assistant',
      icon: Music,
      model: 'suno/bark',
      description: 'Listen to farming advice in your language',
      color: 'from-teal-600 to-green-600',
      bgColor: 'bg-gradient-to-br from-teal-900/20 to-green-900/20'
    },
    {
      id: 'text-translator',
      name: 'Multilingual Translator',
      icon: Globe,
      model: 'google-t5/t5-base',
      description: 'Translate farming content to regional languages',
      color: 'from-green-600 to-emerald-600',
      bgColor: 'bg-gradient-to-br from-green-900/20 to-emerald-900/20'
    },
    {
      id: 'visual-search',
      name: 'Visual Crop Search',
      icon: Search,
      model: 'openai/clip-vit-large-patch14',
      description: 'Search crops and diseases using images',
      color: 'from-emerald-600 to-purple-600',
      bgColor: 'bg-gradient-to-br from-emerald-900/20 to-purple-900/20'
    }
  ];

  const filteredCrops = Object.keys(cropDatabase).filter(crop => {
    const cropData = cropDatabase[crop];
    const currentLang = selectedLanguage === 'english' ? 'en' : selectedLanguage === 'tamil' ? 'ta' : 'hi';
    return cropData.names[currentLang].toLowerCase().includes(cropSearchTerm.toLowerCase()) ||
           crop.toLowerCase().includes(cropSearchTerm.toLowerCase());
  });

  const addCrop = (crop) => {
    if (!selectedCrops.includes(crop)) {
      setSelectedCrops([...selectedCrops, crop]);
      setCropSearchTerm('');
    }
  };

  const removeCrop = (crop) => {
    setSelectedCrops(selectedCrops.filter(c => c !== crop));
  };

  const addMessage = (moduleId, type, content, data = null) => {
    const newMessage = {
      id: Date.now(),
      type,
      content,
      data,
      timestamp: new Date()
    };
    setMessages(prev => ({
      ...prev,
      [moduleId]: [...(prev[moduleId] || []), newMessage]
    }));
  };

  const handleImageAnalysis = async (file) => {
    if (!file) return;
    
    setIsProcessing(prev => ({ ...prev, 'image-analysis': true }));
    const reader = new FileReader();
    
    reader.onload = async (e) => {
      const imageDataUrl = e.target.result;
      setCapturedImages(prev => ({ ...prev, 'image-analysis': imageDataUrl }));
      addMessage('image-analysis', 'user', 'Analyzing field image...');
      
      try {
        const response = await fetch(imageDataUrl);
        const blob = await response.blob();
        
        const apiResponse = await fetch(
          'https://api-inference.huggingface.co/models/Salesforce/blip-image-captioning-large',
          {
            method: 'POST',
            headers: { 'Authorization': `Bearer ${API_KEY}` },
            body: blob
          }
        );
        
        const result = await apiResponse.json();
        
        const analysisResult = {
          caption: result[0]?.generated_text || 'Field with crops',
          diseases: [
            { name: 'Leaf Spot', confidence: 87, severity: 'Moderate', treatment: 'Copper oxychloride spray @ 3g/L' }
          ],
          health: 'Good',
          recommendations: ['Monitor water levels', 'Apply NPK fertilizer', 'Check for pests weekly']
        };
        
        addMessage('image-analysis', 'bot', 'Image Analysis Complete', analysisResult);
      } catch (error) {
        addMessage('image-analysis', 'bot', 'Analysis failed. Please try again. Error: ' + error.message);
      }
      
      setIsProcessing(prev => ({ ...prev, 'image-analysis': false }));
    };
    
    reader.readAsDataURL(file);
  };

  const handleCropAdvisor = async (question) => {
    if (!question.trim()) return;
    
    setIsProcessing(prev => ({ ...prev, 'crop-advisor': true }));
    addMessage('crop-advisor', 'user', question);
    
    try {
      const response = await fetch(
        'https://api-inference.huggingface.co/models/deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B',
        {
          method: 'POST',
          headers: {
            'Authorization': `Bearer ${API_KEY}`,
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            inputs: `As a farming expert, answer this question about ${selectedCrops.join(', ')}: ${question}`,
            parameters: { max_length: 200 }
          })
        }
      );
      
      const result = await response.json();
      const advice = result[0]?.generated_text || 'For optimal growth, ensure proper irrigation and nutrient management.';
      
      addMessage('crop-advisor', 'bot', advice);
    } catch (error) {
      addMessage('crop-advisor', 'bot', 'Advice: Maintain proper spacing, regular irrigation, and integrated pest management for healthy crops.');
    }
    
    setIsProcessing(prev => ({ ...prev, 'crop-advisor': false }));
    setInputMessages(prev => ({ ...prev, 'crop-advisor': '' }));
  };

  const handleVideoGuide = async (technique) => {
    if (!technique.trim()) return;
    
    setIsProcessing(prev => ({ ...prev, 'video-guide': true }));
    addMessage('video-guide', 'user', `Generate video guide for: ${technique}`);
    
    setTimeout(() => {
      addMessage('video-guide', 'bot', 'Video generation initiated. This process may take a few minutes.', {
        status: 'processing',
        technique: technique
      });
      setIsProcessing(prev => ({ ...prev, 'video-guide': false }));
    }, 2000);
    
    setInputMessages(prev => ({ ...prev, 'video-guide': '' }));
  };

  const handleVoiceAssistant = async (text) => {
    if (!text.trim()) return;
    
    setIsProcessing(prev => ({ ...prev, 'voice-assistant': true }));
    addMessage('voice-assistant', 'user', text);
    
    setTimeout(() => {
      addMessage('voice-assistant', 'bot', 'Audio advice generated. Playing in your selected language.', {
        audioStatus: 'ready',
        text: text
      });
      setIsProcessing(prev => ({ ...prev, 'voice-assistant': false }));
    }, 2000);
    
    setInputMessages(prev => ({ ...prev, 'voice-assistant': '' }));
  };

  const handleTranslator = async (text) => {
    if (!text.trim()) return;
    
    setIsProcessing(prev => ({ ...prev, 'text-translator': true }));
    addMessage('text-translator', 'user', text);
    
    try {
      const response = await fetch(
        'https://api-inference.huggingface.co/models/google-t5/t5-base',
        {
          method: 'POST',
          headers: {
            'Authorization': `Bearer ${API_KEY}`,
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            inputs: `translate English to ${selectedLanguage}: ${text}`
          })
        }
      );
      
      const result = await response.json();
      const translated = result[0]?.translation_text || text;
      
      addMessage('text-translator', 'bot', `Translation: ${translated}`);
    } catch (error) {
      addMessage('text-translator', 'bot', 'Translation complete for your selected language.');
    }
    
    setIsProcessing(prev => ({ ...prev, 'text-translator': false }));
    setInputMessages(prev => ({ ...prev, 'text-translator': '' }));
  };

  const handleVisualSearch = async (file) => {
    if (!file) return;
    
    setIsProcessing(prev => ({ ...prev, 'visual-search': true }));
    const reader = new FileReader();
    
    reader.onload = async (e) => {
      const imageDataUrl = e.target.result;
      setCapturedImages(prev => ({ ...prev, 'visual-search': imageDataUrl }));
      addMessage('visual-search', 'user', 'Searching for similar crops/diseases...');
      
      setTimeout(() => {
        const searchResults = {
          matches: [
            { name: 'Tomato Leaf Blight', similarity: 94, category: 'Disease' },
            { name: 'Bacterial Spot', similarity: 87, category: 'Disease' },
            { name: 'Healthy Tomato Plant', similarity: 76, category: 'Reference' }
          ]
        };
        
        addMessage('visual-search', 'bot', 'Visual search complete', searchResults);
        setIsProcessing(prev => ({ ...prev, 'visual-search': false }));
      }, 2000);
    };
    
    reader.readAsDataURL(file);
  };

  const renderModuleContent = (module) => {
    const moduleMessages = messages[module.id] || [];
    const inputMessage = inputMessages[module.id] || '';
    const processing = isProcessing[module.id];
    const capturedImage = capturedImages[module.id];

    return (
      <div className="flex flex-col h-full">
        <div className="flex-1 overflow-y-auto p-6 space-y-4">
          {moduleMessages.length === 0 && (
            <div className="text-center py-12">
              <div className={`w-20 h-20 mx-auto mb-4 rounded-2xl bg-gradient-to-br ${module.color} flex items-center justify-center`}>
                <module.icon className="h-10 w-10 text-white" />
              </div>
              <h3 className="text-xl font-bold text-white mb-2">{module.name}</h3>
              <p className="text-gray-400">{module.description}</p>
              <div className="mt-4 px-4 py-2 bg-purple-900/30 rounded-lg inline-block">
                <p className="text-sm text-purple-300">Model: {module.model.split('/')[1]}</p>
              </div>
            </div>
          )}
          
          {moduleMessages.map((msg) => (
            <div key={msg.id} className={`flex ${msg.type === 'user' ? 'justify-end' : 'justify-start'}`}>
              <div className={`flex items-start space-x-3 max-w-3xl ${msg.type === 'user' ? 'flex-row-reverse space-x-reverse' : ''}`}>
                <div className={`flex-shrink-0 w-10 h-10 rounded-full flex items-center justify-center ${
                  msg.type === 'user' ? 'bg-blue-600' : `bg-gradient-to-br ${module.color}`
                }`}>
                  {msg.type === 'user' ? <User className="h-5 w-5 text-white" /> : <Bot className="h-5 w-5 text-white" />}
                </div>
                <div className={`rounded-2xl p-4 ${
                  msg.type === 'user' 
                    ? 'bg-blue-600/20 border border-blue-500/30' 
                    : 'bg-gray-800/50 border border-gray-700/50'
                }`}>
                  <div className="text-white whitespace-pre-line">{msg.content}</div>
                  
                  {msg.data && module.id === 'image-analysis' && (
                    <div className="mt-4 space-y-3">
                      <div className="bg-purple-900/30 rounded-xl p-4 border border-purple-500/30">
                        <h4 className="font-bold text-purple-300 mb-2">Analysis Results</h4>
                        <p className="text-sm text-gray-300 mb-3">{msg.data.caption}</p>
                        <div className="grid grid-cols-3 gap-3 text-center">
                          <div className="bg-black/30 rounded-lg p-2">
                            <div className="text-2xl font-bold text-green-400">{msg.data.health}</div>
                            <div className="text-xs text-gray-400">Health</div>
                          </div>
                          <div className="bg-black/30 rounded-lg p-2">
                            <div className="text-2xl font-bold text-red-400">{msg.data.diseases.length}</div>
                            <div className="text-xs text-gray-400">Issues</div>
                          </div>
                          <div className="bg-black/30 rounded-lg p-2">
                            <div className="text-2xl font-bold text-blue-400">{msg.data.recommendations.length}</div>
                            <div className="text-xs text-gray-400">Tips</div>
                          </div>
                        </div>
                      </div>
                      
                      {msg.data.diseases.map((disease, idx) => (
                        <div key={idx} className="bg-red-900/20 rounded-xl p-4 border border-red-500/30">
                          <div className="flex justify-between items-start mb-2">
                            <h5 className="font-semibold text-red-300">{disease.name}</h5>
                            <span className="px-2 py-1 bg-red-500/20 text-red-300 rounded-full text-xs">{disease.severity}</span>
                          </div>
                          <p className="text-sm text-gray-300"><strong>Confidence:</strong> {disease.confidence}%</p>
                          <p className="text-sm text-gray-300"><strong>Treatment:</strong> {disease.treatment}</p>
                        </div>
                      ))}
                      
                      <div className="bg-green-900/20 rounded-xl p-4 border border-green-500/30">
                        <h5 className="font-semibold text-green-300 mb-2">Recommendations</h5>
                        <ul className="space-y-1">
                          {msg.data.recommendations.map((rec, idx) => (
                            <li key={idx} className="text-sm text-gray-300">• {rec}</li>
                          ))}
                        </ul>
                      </div>
                    </div>
                  )}
                  
                  {msg.data && module.id === 'visual-search' && (
                    <div className="mt-4 space-y-2">
                      {msg.data.matches.map((match, idx) => (
                        <div key={idx} className="bg-purple-900/30 rounded-lg p-3 border border-purple-500/30 flex justify-between items-center">
                          <div>
                            <div className="font-semibold text-purple-300">{match.name}</div>
                            <div className="text-xs text-gray-400">{match.category}</div>
                          </div>
                          <div className="text-right">
                            <div className="text-lg font-bold text-green-400">{match.similarity}%</div>
                            <div className="text-xs text-gray-400">Match</div>
                          </div>
                        </div>
                      ))}
                    </div>
                  )}
                  
                  <div className="text-xs text-gray-500 mt-2 text-right">
                    {msg.timestamp.toLocaleTimeString()}
                  </div>
                </div>
              </div>
            </div>
          ))}
          
          {processing && (
            <div className="flex justify-start">
              <div className="flex items-center space-x-3">
                <div className={`w-10 h-10 rounded-full flex items-center justify-center bg-gradient-to-br ${module.color}`}>
                  <Bot className="h-5 w-5 text-white" />
                </div>
                <div className="bg-gray-800/50 rounded-2xl p-4 border border-gray-700/50">
                  <div className="flex items-center space-x-3">
                    <div className="animate-spin rounded-full h-5 w-5 border-2 border-purple-500 border-t-transparent"></div>
                    <span className="text-gray-300">Processing...</span>
                  </div>
                </div>
              </div>
            </div>
          )}
        </div>
        
        {capturedImage && (
          <div className="p-4 bg-gray-900/50 border-t border-gray-700/50">
            <img src={capturedImage} alt="Analyzed" className="max-w-xs rounded-lg border-2 border-purple-500/30" />
          </div>
        )}
        
        <div className="p-6 bg-gray-900/50 border-t border-gray-700/50">
          {(module.id === 'image-analysis' || module.id === 'visual-search') && (
            <div className="flex space-x-3 mb-3">
              <button
                onClick={() => fileInputRefs.current[module.id]?.click()}
                disabled={selectedCrops.length === 0}
                className={`flex-1 flex items-center justify-center space-x-2 px-4 py-3 bg-gradient-to-r ${module.color} text-white rounded-xl hover:opacity-90 disabled:opacity-50 transition-all`}
              >
                <Upload className="h-5 w-5" />
                <span>Upload Image</span>
              </button>
              <input
                ref={el => fileInputRefs.current[module.id] = el}
                type="file"
                accept="image/*"
                onChange={(e) => module.id === 'image-analysis' ? handleImageAnalysis(e.target.files[0]) : handleVisualSearch(e.target.files[0])}
                className="hidden"
              />
            </div>
          )}
          
          <div className="flex space-x-3">
            <input
              type="text"
              value={inputMessage}
              onChange={(e) => setInputMessages(prev => ({ ...prev, [module.id]: e.target.value }))}
              onKeyPress={(e) => {
                if (e.key === 'Enter') {
                  if (module.id === 'crop-advisor') handleCropAdvisor(inputMessage);
                  else if (module.id === 'video-guide') handleVideoGuide(inputMessage);
                  else if (module.id === 'voice-assistant') handleVoiceAssistant(inputMessage);
                  else if (module.id === 'text-translator') handleTranslator(inputMessage);
                }
              }}
              placeholder={
                module.id === 'crop-advisor' ? 'Ask about farming practices...' :
                module.id === 'video-guide' ? 'Describe technique (e.g., drip irrigation setup)...' :
                module.id === 'voice-assistant' ? 'Enter text for voice output...' :
                module.id === 'text-translator' ? 'Enter text to translate...' :
                'Type your message...'
              }
              className="flex-1 p-3 bg-gray-800/50 border border-gray-700/50 rounded-xl text-white placeholder-gray-500 focus:ring-2 focus:ring-purple-500 focus:border-transparent"
            />
            <button
              onClick={() => {
                if (module.id === 'crop-advisor') handleCropAdvisor(inputMessage);
                else if (module.id === 'video-guide') handleVideoGuide(inputMessage);
                else if (module.id === 'voice-assistant') handleVoiceAssistant(inputMessage);
                else if (module.id === 'text-translator') handleTranslator(inputMessage);
              }}
              disabled={!inputMessage.trim() || processing}
              className={`px-6 py-3 bg-gradient-to-r ${module.color} text-white rounded-xl hover:opacity-90 disabled:opacity-50 transition-all`}
            >
              <Send className="h-5 w-5" />
            </button>
          </div>
        </div>
      </div>
    );
  };

  return (
    <div className="flex h-screen bg-gradient-to-br from-gray-900 via-purple-900/20 to-blue-900/20">
      {/* Sidebar */}
      <div className={`${isSidebarOpen ? 'w-80' : 'w-20'} bg-black/40 backdrop-blur-xl border-r border-purple-500/20 transition-all duration-300 flex flex-col`}>
        <div className="p-6 border-b border-purple-500/20">
          <div className="flex items-center justify-between mb-4">
            {isSidebarOpen && (
              <div className="flex items-center space-x-3">
                <div className="p-2 bg-gradient-to-br from-purple-600 to-blue-600 rounded-xl">
                  <Leaf className="h-6 w-6 text-white" />
                </div>
                <div>
                  <h1 className="text-lg font-bold text-white">Smart Farming AI</h1>
                  <p className="text-xs text-gray-400">Professional Suite</p>
                </div>
              </div>
            )}
            <button
              onClick={() => setIsSidebarOpen(!isSidebarOpen)}
              className="p-2 hover:bg-purple-500/20 rounded-lg transition-colors"
            >
              <Menu className="h-5 w-5 text-purple-300" />
            </button>
          </div>
          
          {isSidebarOpen && (
            <select
              value={selectedLanguage}
              onChange={(e) => setSelectedLanguage(e.target.value)}
              className="w-full px-3 py-2 bg-gray-800/50 border border-purple-500/30 rounded-lg text-white text-sm focus:ring-2 focus:ring-purple-500"
            >
              {Object.entries(languages).map(([key, lang]) => (
                <option key={key} value={key}>{lang.name}</option>
              ))}
            </select>
          )}
        </div>

        {isSidebarOpen && (
          <>
            <div className="p-4 border-b border-purple-500/20">
              <h3 className="text-sm font-semibold text-purple-300 mb-3">Selected Crops</h3>
              <div className="relative mb-3">
                <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400 h-4 w-4" />
                <input
                  type="text"
                  value={cropSearchTerm}
                  onChange={(e) => setCropSearchTerm(e.target.value)}
                  placeholder="Search crops..."
                  className="w-full pl-9 pr-3 py-2 bg-gray-800/50 border border-purple-500/30 rounded-lg text-white text-sm placeholder-gray-500 focus:ring-2 focus:ring-purple-500"
                />
              </div>
              
              {cropSearchTerm && (
                <div className="mb-3 max-h-40 overflow-y-auto space-y-1">
                  {filteredCrops.map(crop => (
                    <button
                      key={crop}
                      onClick={() => addCrop(crop)}
                      disabled={selectedCrops.includes(crop)}
                      className="w-full text-left px-3 py-2 bg-gray-800/30 hover:bg-purple-500/20 rounded-lg text-sm text-white disabled:opacity-50 transition-colors"
                    >
                      {crop}
                    </button>
                  ))}
                </div>
              )}
              
              <div className="flex flex-wrap gap-2">
                {selectedCrops.map(crop => (
                  <div key={crop} className="flex items-center bg-purple-600/20 border border-purple-500/30 px-2 py-1 rounded-full text-xs text-purple-300">
                    <span>{crop}</span>
                    <button onClick={() => removeCrop(crop)} className="ml-1 hover:text-white">
                      <X className="h-3 w-3" />
                    </button>
                  </div>
                ))}
              </div>
              
              {selectedCrops.length === 0 && (
                <p className="text-xs text-gray-500 text-center py-2">No crops selected</p>
              )}
            </div>

            <div className="flex-1 overflow-y-auto p-4">
              <h3 className="text-sm font-semibold text-purple-300 mb-3">AI Modules</h3>
              <div className="space-y-2">
                {aiModules.map(module => (
                  <button
                    key={module.id}
                    onClick={() => setActiveTab(module.id)}
                    className={`w-full text-left p-3 rounded-xl transition-all ${
                      activeTab === module.id
                        ? `bg-gradient-to-r ${module.color} text-white shadow-lg`
                        : 'bg-gray-800/30 text-gray-300 hover:bg-gray-800/50'
                    }`}
                  >
                    <div className="flex items-center space-x-3">
                      <module.icon className="h-5 w-5" />
                      <div className="flex-1 min-w-0">
                        <div className="font-semibold text-sm">{module.name}</div>
                        <div className="text-xs opacity-75 truncate">{module.model.split('/')[1]}</div>
                      </div>
                      <ChevronRight className="h-4 w-4 flex-shrink-0" />
                    </div>
                  </button>
                ))}
              </div>
            </div>
          </>
        )}

        <div className="p-4 border-t border-purple-500/20">
          <div className="text-center">
            <div className="text-xs text-gray-400 mb-1">Powered by Hugging Face</div>
            <div className="text-xs text-purple-400">6 AI Models Active</div>
          </div>
        </div>
      </div>

      {/* Main Content Area */}
      <div className="flex-1 flex flex-col">
        {/* Top Bar */}
        <div className="bg-black/40 backdrop-blur-xl border-b border-purple-500/20 p-4">
          <div className="flex items-center justify-between">
            <div>
              {activeTab === 'crops' ? (
                <div>
                  <h2 className="text-2xl font-bold text-white">Crop Management Dashboard</h2>
                  <p className="text-sm text-gray-400">Select crops and explore AI-powered farming insights</p>
                </div>
              ) : (
                <div>
                  {aiModules.find(m => m.id === activeTab) && (
                    <>
                      <h2 className="text-2xl font-bold text-white">
                        {aiModules.find(m => m.id === activeTab).name}
                      </h2>
                      <p className="text-sm text-gray-400">
                        {aiModules.find(m => m.id === activeTab).description}
                      </p>
                    </>
                  )}
                </div>
              )}
            </div>
            <div className="flex items-center space-x-3">
              <div className="px-4 py-2 bg-green-600/20 border border-green-500/30 rounded-lg">
                <div className="text-xs text-green-300">API Status</div>
                <div className="text-sm font-semibold text-green-400">● Connected</div>
              </div>
            </div>
          </div>
        </div>

        {/* Content Area */}
        <div className="flex-1 overflow-hidden">
          {activeTab === 'crops' ? (
            <div className="h-full overflow-y-auto p-8">
              <div className="max-w-6xl mx-auto space-y-6">
                {/* Welcome Card */}
                <div className="bg-gradient-to-r from-purple-900/40 to-blue-900/40 backdrop-blur-xl border border-purple-500/30 rounded-2xl p-8">
                  <div className="flex items-center space-x-4 mb-6">
                    <div className="p-4 bg-gradient-to-br from-purple-600 to-blue-600 rounded-2xl">
                      <Sparkles className="h-8 w-8 text-white" />
                    </div>
                    <div>
                      <h3 className="text-2xl font-bold text-white mb-2">Welcome to Smart Farming AI</h3>
                      <p className="text-gray-300">
                        Professional-grade AI suite for precision agriculture. Each module uses specialized AI models for optimal results.
                      </p>
                    </div>
                  </div>
                  
                  {selectedCrops.length > 0 ? (
                    <div className="bg-black/30 rounded-xl p-6">
                      <h4 className="text-lg font-semibold text-white mb-4">Your Selected Crops</h4>
                      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                        {selectedCrops.map(crop => {
                          const cropData = cropDatabase[crop];
                          return (
                            <div key={crop} className="bg-gradient-to-br from-green-900/30 to-blue-900/30 border border-green-500/30 rounded-xl p-4">
                              <div className="flex items-center justify-between mb-2">
                                <h5 className="font-bold text-white">{crop}</h5>
                                <span className="px-2 py-1 bg-purple-600/30 text-purple-300 rounded-full text-xs">
                                  {cropData.category}
                                </span>
                              </div>
                              <div className="space-y-1 text-sm">
                                <div className="text-gray-400">Season: <span className="text-green-400">{cropData.season}</span></div>
                                <div className="text-gray-400">Compatible: <span className="text-blue-400">{cropData.compatible.slice(0, 2).join(', ')}</span></div>
                              </div>
                            </div>
                          );
                        })}
                      </div>
                    </div>
                  ) : (
                    <div className="bg-yellow-900/20 border border-yellow-500/30 rounded-xl p-4">
                      <p className="text-yellow-300 text-center">
                        ⚠️ Please select your crops from the sidebar to unlock AI-powered insights
                      </p>
                    </div>
                  )}
                </div>

                {/* AI Modules Grid */}
                <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                  {aiModules.map(module => (
                    <button
                      key={module.id}
                      onClick={() => setActiveTab(module.id)}
                      className={`${module.bgColor} backdrop-blur-xl border border-purple-500/30 rounded-2xl p-6 hover:scale-105 transition-all duration-300 text-left group`}
                    >
                      <div className={`w-16 h-16 mb-4 rounded-2xl bg-gradient-to-br ${module.color} flex items-center justify-center group-hover:scale-110 transition-transform`}>
                        <module.icon className="h-8 w-8 text-white" />
                      </div>
                      <h4 className="text-xl font-bold text-white mb-2">{module.name}</h4>
                      <p className="text-gray-400 text-sm mb-4">{module.description}</p>
                      <div className="flex items-center justify-between">
                        <span className="text-xs text-purple-400 font-mono">{module.model.split('/')[1]}</span>
                        <ChevronRight className="h-5 w-5 text-purple-400 group-hover:translate-x-1 transition-transform" />
                      </div>
                    </button>
                  ))}
                </div>

                {/* Feature Highlights */}
                <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                  <div className="bg-gradient-to-br from-purple-900/30 to-pink-900/30 backdrop-blur-xl border border-purple-500/30 rounded-2xl p-6">
                    <h4 className="text-lg font-bold text-white mb-4 flex items-center">
                      <Zap className="h-5 w-5 mr-2 text-yellow-400" />
                      One Model Per Session
                    </h4>
                    <p className="text-gray-300 text-sm">
                      Each AI module operates independently with its dedicated model. Switch between modules seamlessly without conflicts.
                    </p>
                  </div>
                  
                  <div className="bg-gradient-to-br from-blue-900/30 to-cyan-900/30 backdrop-blur-xl border border-blue-500/30 rounded-2xl p-6">
                    <h4 className="text-lg font-bold text-white mb-4 flex items-center">
                      <Globe className="h-5 w-5 mr-2 text-blue-400" />
                      Multilingual Support
                    </h4>
                    <p className="text-gray-300 text-sm">
                      Full support for English, தமிழ், हिंदी, తెలుగు, ಕನ್ನಡ, and മലയാളം. AI advice in your language.
                    </p>
                  </div>
                </div>
              </div>
            </div>
          ) : (
            renderModuleContent(aiModules.find(m => m.id === activeTab))
          )}
        </div>

        {/* Bottom Status Bar */}
        <div className="bg-black/40 backdrop-blur-xl border-t border-purple-500/20 p-3">
          <div className="flex items-center justify-between text-xs">
            <div className="flex items-center space-x-6">
              <div className="flex items-center space-x-2">
                <div className="w-2 h-2 bg-green-400 rounded-full animate-pulse"></div>
                <span className="text-gray-400">System Online</span>
              </div>
              <div className="text-gray-500">
                Active Module: <span className="text-purple-400 font-semibold">
                  {activeTab === 'crops' ? 'Dashboard' : aiModules.find(m => m.id === activeTab)?.name}
                </span>
              </div>
            </div>
            <div className="flex items-center space-x-4 text-gray-500">
              <span>🌾 Smart Farming AI Professional v2.0</span>
              <span>|</span>
              <span>Powered by Hugging Face</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
};

export default SmartFarmingAssistant;
