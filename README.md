<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SMEG - Design & Innovation</title>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,700;1,400&family=Lato:wght@300;400;700&display=swap" rel="stylesheet">
    
    <!-- React & ReactDOM -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    
    <!-- Babel for JSX -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>

    <style>
        body {
            font-family: 'Lato', sans-serif;
            background-color: #F9F9F7;
        }
        h1, h2, h3, .serif {
            font-family: 'Playfair Display', serif;
        }
        
        /* Animation du scanner */
        @keyframes scanline {
            0% { top: 0%; opacity: 0; }
            10% { opacity: 1; }
            90% { opacity: 1; }
            100% { top: 100%; opacity: 0; }
        }
        .scan-animation {
            position: absolute;
            width: 100%;
            height: 4px;
            background: #3B82F6;
            box-shadow: 0 0 15px #3B82F6;
            animation: scanline 3s infinite linear;
            z-index: 10;
        }
        
        /* Energy Label Colors */
        .energy-A { background-color: #009640; }
        .energy-B { background-color: #5bbd2b; }
        .energy-C { background-color: #cddc00; }
        .energy-D { background-color: #ffde00; }
        .energy-E { background-color: #fbba00; }
        .energy-F { background-color: #eb7c00; }
        .energy-G { background-color: #e30613; }

        /* Animation d'apparition */
        @keyframes fade-in-up {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .animate-fade-in-up {
            animation: fade-in-up 0.8s ease-out forwards;
        }

        .editable-image-container:hover .edit-overlay {
            opacity: 1;
        }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useRef } = React;

        // --- Composant Image Éditable (Click & Paste) ---
        // Ajout de la prop 'readOnly' pour désactiver l'édition en mode présentation
        const EditableImage = ({ src, alt, className, imgClassName, onImageChange, style, readOnly }) => {
            const fileInputRef = useRef(null);
            const containerRef = useRef(null);

            // Si en mode lecture seule (Présentation), on rend juste l'image simple
            if (readOnly) {
                return (
                    <div className={`relative ${className}`} style={style}>
                        <img 
                            src={src} 
                            alt={alt} 
                            className={imgClassName || "w-full h-full object-cover"} 
                        />
                    </div>
                );
            }

            const handleDivClick = (e) => {
                e.stopPropagation();
                fileInputRef.current.click();
            };

            const handleFileChange = (e) => {
                const file = e.target.files[0];
                if (file) {
                    const imageUrl = URL.createObjectURL(file);
                    onImageChange(imageUrl);
                }
            };

            const handlePaste = (e) => {
                const items = e.clipboardData.items;
                for (let i = 0; i < items.length; i++) {
                    if (items[i].type.indexOf("image") !== -1) {
                        const blob = items[i].getAsFile();
                        const imageUrl = URL.createObjectURL(blob);
                        onImageChange(imageUrl);
                        e.preventDefault(); 
                        return; 
                    }
                }
            };

            return (
                <div 
                    ref={containerRef}
                    className={`relative editable-image-container cursor-pointer group outline-none focus:ring-2 focus:ring-red-500 ${className}`} 
                    onClick={handleDivClick}
                    onPaste={handlePaste}
                    tabIndex="0"
                    style={style}
                    title="Mode Édition : Cliquez ou Ctrl+V pour changer l'image"
                >
                    <img 
                        src={src} 
                        alt={alt} 
                        className={imgClassName || "w-full h-full object-cover"} 
                    />
                    
                    {/* Overlay d'édition */}
                    <div className="edit-overlay absolute inset-0 bg-black/40 opacity-0 transition-opacity duration-300 flex flex-col items-center justify-center z-20 pointer-events-none">
                        <div className="bg-white/90 text-black px-4 py-2 rounded-full flex items-center gap-2 text-xs font-bold uppercase tracking-wider shadow-lg transform group-hover:scale-105 transition mb-2">
                            <i data-lucide="upload" className="w-4 h-4"></i> Changer la photo
                        </div>
                        <div className="text-white text-[10px] font-mono bg-black/50 px-2 py-1 rounded">
                            ou Ctrl+V pour coller
                        </div>
                    </div>

                    <input 
                        type="file" 
                        ref={fileInputRef} 
                        onChange={handleFileChange} 
                        className="hidden" 
                        accept="image/*"
                    />
                </div>
            );
        };

        const EnergyLabel = ({ grade, consumption }) => {
            const grades = ['A', 'B', 'C', 'D', 'E', 'F', 'G'];
            return (
                <div className="mt-4 p-4 bg-gray-50 rounded-lg border border-gray-100">
                    <div className="flex items-center justify-between mb-2">
                        <span className="text-xs font-bold text-gray-500 uppercase">Efficacité Énergétique</span>
                        <span className="text-xs font-bold text-gray-900">{consumption}</span>
                    </div>
                    <div className="flex flex-col gap-1 w-full max-w-[200px]">
                        {grades.map((g) => {
                            const isActive = g === grade;
                            return (
                                <div key={g} className="flex items-center h-4 relative">
                                    {isActive && (
                                        <div className="absolute left-full ml-2 flex items-center z-10 w-max">
                                            <div className="bg-black text-white text-xs font-bold px-1.5 py-0.5 rounded-sm shadow-md">
                                                {g}
                                            </div>
                                            <div className="w-0 h-0 border-t-[4px] border-t-transparent border-r-[6px] border-r-black border-b-[4px] border-b-transparent -ml-1"></div>
                                        </div>
                                    )}
                                    <div 
                                        className={`h-full rounded-r-sm energy-${g} opacity-${isActive ? '100' : '30'}`} 
                                        style={{ width: `${30 + (grades.indexOf(g) * 8)}px` }}
                                    ></div>
                                </div>
                            )
                        })}
                    </div>
                </div>
            );
        };

        const ScannerInterface = ({ onScanComplete, cameraImage }) => {
            const [scanning, setScanning] = useState(true);
            const [progress, setProgress] = useState(0);
            const [message, setMessage] = useState("Initialisation de la caméra...");
            const [detected, setDetected] = useState(null);

            useEffect(() => {
                const interval = setInterval(() => {
                    setProgress(prev => {
                        if (prev >= 100) {
                            clearInterval(interval);
                            setScanning(false);
                            setDetected({ w: 90, h: 200, d: 80, space: "Mur Nord" }); 
                            return 100;
                        }
                        if (prev === 30) setMessage("Analyse de la profondeur...");
                        if (prev === 60) setMessage("Détection des surfaces...");
                        if (prev === 90) setMessage("Mesure précise...");
                        return prev + 1;
                    });
                }, 40);
                return () => clearInterval(interval);
            }, []);

            return (
                <div className="fixed inset-0 bg-black z-50 flex flex-col text-white">
                    <div className="absolute inset-0 bg-gray-900">
                         <img 
                            src={cameraImage}
                            className="w-full h-full object-cover opacity-50"
                            alt="Camera Feed"
                         />
                    </div>
                    
                    <div className="relative z-10 flex flex-col h-full p-6 justify-between">
                        <div className="flex justify-between items-start">
                            <button onClick={() => onScanComplete(null)} className="p-2 bg-white/10 backdrop-blur rounded-full hover:bg-white/20">
                                <i data-lucide="x" className="w-6 h-6"></i>
                            </button>
                            <div className="bg-red-600 px-3 py-1 rounded text-xs font-bold uppercase animate-pulse shadow-lg">
                                Live Scan
                            </div>
                        </div>

                        <div className="absolute inset-0 flex items-center justify-center pointer-events-none">
                            <div className="w-3/4 h-1/2 border-2 border-white/30 rounded-lg relative overflow-hidden transition-all duration-500" style={{borderColor: detected ? '#4ade80' : 'rgba(255,255,255,0.3)'}}>
                                {scanning && <div className="scan-animation"></div>}
                                
                                {!scanning && detected && (
                                    <div className="absolute inset-0 flex items-center justify-center bg-black/60 backdrop-blur-sm flex-col pointer-events-auto animate-fade-in-up">
                                        <div className="bg-white text-black p-6 rounded-lg text-center shadow-2xl max-w-sm mx-4">
                                            <div className="text-green-500 mb-2 flex justify-center"><i data-lucide="check-circle" className="w-12 h-12"></i></div>
                                            <h3 className="serif text-2xl font-bold mb-1">Espace validé !</h3>
                                            <p className="text-sm text-gray-500 mb-4">Parfait pour un Réfrigérateur FAB50.</p>
                                            <button 
                                                onClick={() => onScanComplete(detected)}
                                                className="w-full bg-black text-white py-4 rounded-lg uppercase text-sm font-bold hover:bg-red-600 transition shadow-lg flex items-center justify-center gap-2"
                                            >
                                                <span>Voir le résultat</span>
                                                <i data-lucide="arrow-right" className="w-4 h-4"></i>
                                            </button>
                                        </div>
                                    </div>
                                )}
                            </div>
                        </div>
                         <div className="text-center mb-10">
                            {scanning ? (
                                <div>
                                    <p className="text-lg font-medium mb-2 drop-shadow-md">{message}</p>
                                    <div className="w-64 h-1 bg-gray-700 mx-auto rounded overflow-hidden">
                                        <div className="h-full bg-blue-500 transition-all duration-300" style={{width: `${progress}%`}}></div>
                                    </div>
                                </div>
                            ) : null}
                        </div>
                    </div>
                </div>
            );
        };

        const ProductCard = ({ product, scannedData, onSelect, onImageUpdate, readOnly }) => {
            const [selectedColor, setSelectedColor] = useState(product.colors[0]);
            const fits = scannedData ? (product.dimensions.w <= scannedData.w) : true;

            return (
                <div className="group relative bg-white rounded-xl shadow-sm hover:shadow-xl transition-all duration-500 overflow-hidden border border-gray-100 flex flex-col h-full">
                    
                    <div className="relative h-72 bg-white flex items-center justify-center overflow-hidden p-4">
                        
                        <div className="w-full h-full relative z-20">
                            <EditableImage 
                                src={product.image} 
                                alt={product.name} 
                                className="w-full h-full"
                                imgClassName="w-full h-full object-contain transition-transform duration-700 group-hover:scale-105"
                                onImageChange={(newUrl) => onImageUpdate(product.id, newUrl)}
                                readOnly={readOnly}
                            />
                        </div>

                        {fits && scannedData && (
                            <div className="absolute top-3 right-3 z-30 bg-green-100 text-green-700 text-xs font-bold px-3 py-1 rounded-full flex items-center gap-1 shadow-sm pointer-events-none">
                                <i data-lucide="check" className="w-3 h-3"></i> Compatible
                            </div>
                        )}
                    </div>

                    <div className="p-6 flex flex-col flex-grow relative z-10 border-t border-gray-50" onClick={() => onSelect({...product, selectedColor})}>
                        <div className="flex justify-between items-start mb-2">
                            <div>
                                <p className="text-xs text-gray-500 uppercase tracking-widest mb-1">{product.type}</p>
                                <h3 className="serif text-xl font-bold text-gray-900 leading-tight">{product.name}</h3>
                            </div>
                            <span className="font-mono font-bold text-lg text-red-600">{product.price}€</span>
                        </div>
                        
                        <p className="text-sm text-gray-500 mb-4 line-clamp-2 flex-grow">{product.desc}</p>
                        
                        <div className="mb-4" onClick={(e) => e.stopPropagation()}>
                            <div className="flex gap-2">
                                {product.colors.map((color) => (
                                    <button
                                        key={color.name}
                                        onClick={() => setSelectedColor(color)}
                                        className={`w-6 h-6 rounded-full border-2 transition-all shadow-sm ${selectedColor.name === color.name ? 'border-gray-900 scale-110 ring-1 ring-gray-300' : 'border-white hover:scale-110'}`}
                                        style={{ backgroundColor: color.hex }}
                                        title={color.name}
                                    />
                                ))}
                            </div>
                        </div>

                        <div className="flex items-center justify-between border-t border-gray-100 pt-4 mt-auto">
                            <button 
                                className="text-xs font-bold uppercase tracking-wider text-black hover:text-red-600 transition flex items-center gap-1"
                            >
                                Configurer <i data-lucide="arrow-right" className="w-3 h-3"></i>
                            </button>
                        </div>
                    </div>
                </div>
            );
        };

        const ProductDetailModal = ({ product, onClose, scannedData }) => {
            const [selectedColor, setSelectedColor] = useState(product.selectedColor || product.colors[0]);

            if (!product) return null;

            return (
                <div className="fixed inset-0 z-50 flex items-center justify-center p-4">
                    <div className="absolute inset-0 bg-black/60 backdrop-blur-sm animate-fade-in-up" onClick={onClose}></div>
                    <div className="bg-white rounded-2xl w-full max-w-5xl max-h-[95vh] overflow-y-auto relative z-10 flex flex-col md:flex-row shadow-2xl animate-fade-in-up">
                        <button onClick={onClose} className="absolute top-4 right-4 z-20 p-2 bg-white/80 rounded-full hover:bg-gray-100 transition shadow-sm">
                            <i data-lucide="x" className="w-5 h-5"></i>
                        </button>

                        <div className="w-full md:w-1/2 bg-gray-50 p-8 md:p-12 flex items-center justify-center relative overflow-hidden">
                             <div 
                                className="absolute inset-0 opacity-5 transition-colors duration-700"
                                style={{ backgroundColor: selectedColor.hex }}
                            ></div>
                            <div className="relative z-10 w-full h-full flex flex-col items-center justify-center">
                                <img 
                                    src={product.image} 
                                    alt={product.name} 
                                    className="w-full h-full object-contain max-h-[400px] drop-shadow-2xl" 
                                />
                                {scannedData && (
                                    <div className="mt-8 bg-white/90 backdrop-blur px-4 py-2 rounded-full shadow-lg flex items-center gap-2 text-green-700 border border-green-100">
                                        <i data-lucide="check-circle" className="w-4 h-4"></i>
                                        <span className="text-xs font-bold">Compatible avec votre espace</span>
                                    </div>
                                )}
                            </div>
                        </div>

                        <div className="w-full md:w-1/2 p-8 md:p-12 overflow-y-auto">
                            <div className="mb-6">
                                <div className="flex items-center gap-2 mb-2">
                                    <span className="text-red-600 text-xs font-bold uppercase tracking-widest bg-red-50 px-2 py-1 rounded">{product.type}</span>
                                </div>
                                <h2 className="serif text-4xl font-bold mt-1 mb-2 text-gray-900">{product.name}</h2>
                                <p className="text-lg text-gray-600 border-b border-gray-100 pb-4 mb-4">{product.desc}</p>
                            </div>

                            <div className="mb-8">
                                <p className="text-sm font-bold text-gray-900 uppercase mb-3">Couleur : {selectedColor.name}</p>
                                <div className="flex gap-3">
                                    {product.colors.map((color) => (
                                        <button
                                            key={color.name}
                                            onClick={() => setSelectedColor(color)}
                                            className={`w-10 h-10 rounded-full border-2 transition-all shadow-sm ${selectedColor.name === color.name ? 'border-gray-900 scale-110 ring-2 ring-offset-2 ring-gray-200' : 'border-white hover:scale-110'}`}
                                            style={{ backgroundColor: color.hex }}
                                            title={color.name}
                                        />
                                    ))}
                                </div>
                            </div>
                             <div className="mb-8">
                                <EnergyLabel grade={product.energyClass} consumption={product.consumption} />
                            </div>

                            <div className="flex gap-4">
                                <button className="flex-1 bg-black text-white py-4 rounded-lg font-bold uppercase tracking-wider hover:bg-red-600 transition shadow-lg transform active:scale-95">
                                    Ajouter au panier
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            );
        };

        const App = () => {
            const [view, setView] = useState('home'); 
            const [scannedData, setScannedData] = useState(null);
            const [selectedProduct, setSelectedProduct] = useState(null);
            // ÉTAT POUR LE MODE PRÉSENTATION
            const [isPresentationMode, setIsPresentationMode] = useState(false);

            const [productsList, setProductsList] = useState([
                {
                    id: 1,
                    name: "Réfrigérateur FAB50",
                    type: "Froid",
                    price: 2799,
                    energyClass: "E",
                    consumption: "265 kWh/an",
                    desc: "Le réfrigérateur double porte iconique. Capacité XXL avec l'esthétique Années 50 inimitable.",
                    dimensions: { w: 80, h: 187, d: 76 },
                    colors: [{ name: "Rouge", hex: "#C8102E" }, { name: "Crème", hex: "#F2E8C9" }, { name: "Bleu Azur", hex: "#4B9CD3" }, { name: "Noir", hex: "#1A1A1A" }, { name: "Blanc", hex: "#FFFFFF" }],
                    image: "https://images.unsplash.com/photo-1571175443880-49e1d58b7948?ixlib=rb-4.0.3&auto=format&fit=crop&w=800&q=80"
                },
                {
                    id: 2,
                    name: "Centre de Cuisson Portofino",
                    type: "Cuisson",
                    price: 3299,
                    energyClass: "A+",
                    consumption: "0.89 kWh/cycle",
                    desc: "Inspiré par les couleurs de la Riviera italienne. Performance professionnelle.",
                    dimensions: { w: 90, h: 90, d: 60 },
                    colors: [{ name: "Rouge", hex: "#A81C07" }, { name: "Jaune", hex: "#F3C012" }, { name: "Vert Olive", hex: "#6B7132" }, { name: "Anthracite", hex: "#2E2E2E" }],
                    image: "https://images.unsplash.com/photo-1533038590840-1cde6e668a91?ixlib=rb-4.0.3&auto=format&fit=crop&w=800&q=80"
                },
                {
                    id: 3,
                    name: "Lave-vaisselle 50's Style",
                    type: "Lavage",
                    price: 1199,
                    energyClass: "B",
                    consumption: "54 kWh/100 cycles",
                    desc: "Technologie de lavage de pointe dissimulée derrière une porte galbée rétro.",
                    dimensions: { w: 60, h: 82, d: 60 },
                    colors: [{ name: "Rouge", hex: "#C8102E" }, { name: "Bleu Pastel", hex: "#A3C1AD" }, { name: "Crème", hex: "#F2E8C9" }],
                    image: "https://images.unsplash.com/photo-1584622650111-993a426fbf0a?ixlib=rb-4.0.3&auto=format&fit=crop&w=800&q=80"
                }
            ]);

            const [partnersList, setPartnersList] = useState([
                { id: 1, name: "Studio Architecture", city: "Paris", rating: 5.0, image: "https://images.unsplash.com/photo-1600607686527-6fb886090705?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80" },
                { id: 2, name: "Cuisines & Co", city: "Bordeaux", rating: 4.8, image: "https://images.unsplash.com/photo-1600585154526-990dced4db0d?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80" },
                { id: 3, name: "Riviera Design", city: "Nice", rating: 4.9, image: "https://images.unsplash.com/photo-1600566752355-35792bedcfe1?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80" }
            ]);

            const [heroImage, setHeroImage] = useState("https://images.unsplash.com/photo-1556910103-1c02745a30bf?ixlib=rb-4.0.3&auto=format&fit=crop&w=1600&q=80");
            const [cat1Image, setCat1Image] = useState("https://images.unsplash.com/photo-1571175443880-49e1d58b7948?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80");
            const [cat2Image, setCat2Image] = useState("https://images.unsplash.com/photo-1533038590840-1cde6e668a91?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80");

            useEffect(() => {
                lucide.createIcons();
            }, [view, selectedProduct, productsList, partnersList, isPresentationMode]);

            const updateProductImage = (id, newUrl) => {
                setProductsList(prev => prev.map(p => p.id === id ? { ...p, image: newUrl } : p));
            };

            const updatePartnerImage = (id, newUrl) => {
                setPartnersList(prev => prev.map(p => p.id === id ? { ...p, image: newUrl } : p));
            };

            const handleScanComplete = (data) => {
                setScannedData(data);
                setView('shop');
                if (data) {
                    setTimeout(() => {
                        const defaultProduct = productsList[0];
                        setSelectedProduct({ ...defaultProduct, selectedColor: defaultProduct.colors[0] });
                    }, 600);
                }
            };

            // Navbar avec bouton pour basculer le mode présentation
            const Navbar = ({ onViewChange, currentView }) => (
                <nav className="fixed top-0 w-full z-50 bg-white/95 backdrop-blur-md border-b border-gray-100 transition-all duration-300">
                    <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                        <div className="flex justify-between items-center h-20">
                            <div className="flex-shrink-0 cursor-pointer" onClick={() => onViewChange('home')}>
                                <h1 className="text-3xl font-bold tracking-widest text-gray-900" style={{fontFamily: 'Lato'}}>SMEG</h1>
                            </div>
                            <div className="hidden md:flex space-x-8 items-center">
                                <button onClick={() => onViewChange('home')} className={`text-sm font-medium uppercase tracking-wider hover:text-red-600 transition ${currentView === 'home' ? 'text-red-600' : 'text-gray-500'}`}>Collections</button>
                                <button onClick={() => onViewChange('shop')} className={`text-sm font-medium uppercase tracking-wider hover:text-red-600 transition ${currentView === 'shop' ? 'text-red-600' : 'text-gray-500'}`}>Produits</button>
                                <button onClick={() => onViewChange('partners')} className={`text-sm font-medium uppercase tracking-wider hover:text-red-600 transition ${currentView === 'partners' ? 'text-red-600' : 'text-gray-500'}`}>Partenaires</button>
                            </div>
                            <div className="flex items-center space-x-4">
                                <button onClick={() => onViewChange('scanner')} className="flex items-center gap-2 bg-black text-white px-5 py-2 rounded-full hover:bg-red-600 transition shadow-lg">
                                    <i data-lucide="scan-line" className="w-4 h-4"></i>
                                    <span className="text-xs font-bold uppercase tracking-wider">Scanner</span>
                                </button>
                                
                                {/* Bouton de bascule Mode Présentation (Visible pour Admin/Demo) */}
                                <button 
                                    onClick={() => setIsPresentationMode(!isPresentationMode)} 
                                    className={`p-2 rounded-full transition-colors ${isPresentationMode ? 'bg-green-100 text-green-700' : 'bg-gray-100 text-gray-500 hover:text-black'}`}
                                    title={isPresentationMode ? "Mode Présentation Actif (Cliquez pour éditer)" : "Activer le Mode Présentation (Cacher les outils)"}
                                >
                                    {isPresentationMode ? <i data-lucide="eye"></i> : <i data-lucide="edit-3"></i>}
                                </button>
                            </div>
                        </div>
                    </div>
                </nav>
            );

            return (
                <div className="min-h-screen flex flex-col">
                    <Navbar onViewChange={setView} currentView={view} />
                    
                    <main className="flex-grow pt-20">
                        {view === 'scanner' && (
                            <ScannerInterface onScanComplete={handleScanComplete} cameraImage={heroImage} />
                        )}

                        {view === 'home' && (
                            <>
                                {/* Hero Section avec image modifiable */}
                                <section className="relative h-[85vh] overflow-hidden flex items-center justify-center">
                                    <div className="absolute inset-0">
                                        <EditableImage 
                                            src={heroImage} 
                                            alt="Hero" 
                                            onImageChange={setHeroImage}
                                            className="w-full h-full"
                                            imgClassName="w-full h-full object-cover"
                                            readOnly={isPresentationMode}
                                        />
                                        <div className="absolute inset-0 bg-black/30 pointer-events-none"></div>
                                    </div>
                                    <div className="relative z-10 text-center text-white px-4 animate-fade-in-up pointer-events-none">
                                        <h1 className="serif text-5xl md:text-7xl font-bold mb-6 drop-shadow-lg">Technologie & Style</h1>
                                        <p className="text-xl md:text-2xl font-light mb-8 max-w-2xl mx-auto drop-shadow-md">
                                            L'électroménager qui transforme votre cuisine en œuvre d'art.
                                        </p>
                                        <div className="flex flex-col md:flex-row gap-4 justify-center pointer-events-auto">
                                            <button onClick={() => setView('shop')} className="px-8 py-4 bg-white text-black font-bold uppercase tracking-widest hover:bg-gray-100 transition shadow-lg rounded-sm">
                                                Découvrir
                                            </button>
                                            <button onClick={() => setView('scanner')} className="px-8 py-4 bg-red-600 text-white font-bold uppercase tracking-widest hover:bg-red-700 transition flex items-center justify-center gap-2 shadow-lg rounded-sm">
                                                Scanner
                                            </button>
                                        </div>
                                    </div>
                                </section>

                                {/* Featured Categories avec images modifiables */}
                                <section className="py-20 bg-gray-50">
                                    <div className="max-w-7xl mx-auto px-4 grid grid-cols-1 md:grid-cols-2 gap-8">
                                        <div className="bg-white p-12 rounded-2xl shadow-sm flex flex-col items-start justify-center min-h-[450px] relative overflow-hidden group border border-gray-100">
                                            <div className="relative z-10 max-w-md pointer-events-none">
                                                <span className="text-red-600 font-bold tracking-widest text-xs uppercase mb-3 block">Iconique</span>
                                                <h2 className="serif text-4xl mb-4 font-bold text-gray-900">Réfrigérateurs FAB</h2>
                                                <p className="text-gray-500 mb-8 leading-relaxed">Le style années 50 réinventé.</p>
                                                <button onClick={() => setView('shop')} className="inline-flex items-center gap-2 text-sm font-bold uppercase tracking-wider border-b-2 border-red-600 pb-1 pointer-events-auto">
                                                    Voir les modèles
                                                </button>
                                            </div>
                                            <div className="absolute right-[-50px] bottom-[-50px] w-2/3 h-2/3">
                                                 <EditableImage 
                                                    src={cat1Image} 
                                                    alt="SMEG Fridge" 
                                                    onImageChange={setCat1Image}
                                                    className="w-full h-full"
                                                    imgClassName="w-full h-full object-contain opacity-90 transition transform duration-700 group-hover:scale-110 group-hover:rotate-3"
                                                    readOnly={isPresentationMode}
                                                />
                                            </div>
                                        </div>
                                        <div className="bg-neutral-900 text-white p-12 rounded-2xl shadow-sm flex flex-col items-start justify-center min-h-[450px] relative overflow-hidden group">
                                            <div className="relative z-10 max-w-md pointer-events-none">
                                                <span className="text-gray-400 font-bold tracking-widest text-xs uppercase mb-3 block">Portofino</span>
                                                <h2 className="serif text-4xl mb-4 font-bold">Gamme Portofino</h2>
                                                <p className="text-gray-400 mb-8 leading-relaxed">La robustesse des cuisines professionnelles.</p>
                                                <button onClick={() => setView('shop')} className="inline-flex items-center gap-2 text-sm font-bold uppercase tracking-wider border-b-2 border-white pb-1 pointer-events-auto">
                                                    Configurer
                                                </button>
                                            </div>
                                            <div className="absolute right-0 bottom-0 w-2/3 h-2/3">
                                                <EditableImage 
                                                    src={cat2Image} 
                                                    alt="Portofino" 
                                                    onImageChange={setCat2Image}
                                                    className="w-full h-full"
                                                    imgClassName="w-full h-full object-contain opacity-40 mix-blend-lighten transition transform duration-700 group-hover:scale-105"
                                                    readOnly={isPresentationMode}
                                                />
                                            </div>
                                        </div>
                                    </div>
                                </section>
                            </>
                        )}

                        {view === 'shop' && (
                            <section className="py-12 bg-gray-50 min-h-screen">
                                <div className="max-w-7xl mx-auto px-4">
                                    <div className="mb-12 flex flex-col md:flex-row md:items-end justify-between gap-4">
                                        <div>
                                            <h2 className="serif text-4xl font-bold mb-2 text-gray-900">Configurez votre style</h2>
                                            {!isPresentationMode && <p className="text-gray-500 animate-pulse">📝 Mode Édition : Cliquez ou Ctrl+V sur une image pour la changer.</p>}
                                        </div>
                                    </div>

                                    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
                                        {productsList.map(product => (
                                            <ProductCard 
                                                key={product.id} 
                                                product={product} 
                                                scannedData={scannedData}
                                                onSelect={setSelectedProduct}
                                                onImageUpdate={updateProductImage}
                                                readOnly={isPresentationMode}
                                            />
                                        ))}
                                    </div>
                                </div>
                            </section>
                        )}

                        {(view === 'home' || view === 'partners') && (
                            <div className="py-20 bg-white">
                                <div className="max-w-7xl mx-auto px-4">
                                    <div className="text-center mb-16">
                                        <h2 className="serif text-4xl md:text-5xl font-bold mt-2 mb-4 text-gray-900">Nos Partenaires</h2>
                                        {!isPresentationMode && <p className="text-gray-500">Cliquez ou collez pour changer les photos.</p>}
                                    </div>
                                    <div className="grid grid-cols-1 md:grid-cols-3 gap-8">
                                        {partnersList.map(partner => (
                                            <div key={partner.id} className="group cursor-pointer">
                                                <div className="relative overflow-hidden rounded-lg aspect-[4/3] mb-4 bg-gray-100">
                                                    <EditableImage 
                                                        src={partner.image} 
                                                        alt={partner.name}
                                                        onImageChange={(newUrl) => updatePartnerImage(partner.id, newUrl)}
                                                        className="w-full h-full"
                                                        imgClassName="w-full h-full object-cover transition duration-700 group-hover:scale-105"
                                                        readOnly={isPresentationMode}
                                                    />
                                                    <div className="absolute bottom-0 left-0 bg-white px-4 py-2 rounded-tr-lg shadow-md pointer-events-none">
                                                        <div className="flex items-center gap-1">
                                                            <i data-lucide="star" className="w-3 h-3 fill-yellow-400 text-yellow-400"></i>
                                                            <span className="text-xs font-bold">{partner.rating}</span>
                                                        </div>
                                                    </div>
                                                </div>
                                                <h3 className="serif text-xl font-bold">{partner.name}</h3>
                                                <p className="text-sm text-gray-500 mb-2 flex items-center gap-1">
                                                    <i data-lucide="map-pin" className="w-3 h-3"></i> {partner.city}
                                                </p>
                                            </div>
                                        ))}
                                    </div>
                                </div>
                            </div>
                        )}
                    </main>

                    {selectedProduct && (
                        <ProductDetailModal 
                            product={selectedProduct} 
                            scannedData={scannedData}
                            onClose={() => setSelectedProduct(null)} 
                        />
                    )}
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
