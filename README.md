import React, { useState, useMemo, useRef } from 'react';
import { 
  Upload, Star, MapPin, Car, Ship, User, 
  Calendar, Phone, CheckCircle2, XCircle, Search, Bus,
  Edit2, Trash2, Save, X, AlertCircle
} from 'lucide-react';

// --- MOCK DATA ---
const baseMockData = [
  { id: 1, Name: "JOYCE", Category: "Guide", Status: "Available", Zone: "Near", Rating: "5", Expertise: "Damnoen Floating Market", BookingDate: "2026-05-20", Phone: "095-5392455" },
  { id: 2, Name: "Mr. Odd", Category: "Car", Status: "Busy", Zone: "Far", Rating: "4", Expertise: "Ayutthaya", BookingDate: "2026-05-20", Phone: "097-201-5963" },
  { id: 3, Name: "Mr. Sommai", Category: "Boat", Status: "Available", Zone: "Near", Rating: "5", Expertise: "Chao Phraya River", BookingDate: "", Phone: "081-1234567" },
  { id: 4, Name: "WAN", Category: "Guide", Status: "Busy", Zone: "Near", Rating: "3", Expertise: "Grand Palace", BookingDate: "2026-05-21", Phone: "081-9298077" },
  { id: 5, Name: "K.Nu", Category: "Van", Status: "Available", Zone: "Far", Rating: "5", Expertise: "Pattaya", BookingDate: "", Phone: "098-2810638" },
  { id: 6, Name: "KITTY", Category: "Guide", Status: "Available", Zone: "Far", Rating: "4", Expertise: "Maeklong", BookingDate: "2026-05-25", Phone: "082-1491944" },
  { id: 7, Name: "Boat 99", Category: "Boat", Status: "Busy", Zone: "Far", Rating: "4", Expertise: "Ayutthaya River Cruise", BookingDate: "2026-05-20", Phone: "089-9999999" },
  { id: 8, Name: "SURINA", Category: "Guide", Status: "Available", Zone: "Near", Rating: "5", Expertise: "Food Tour", BookingDate: "", Phone: "081-5858114" },
  { id: 9, Name: "Mai (Phim)", Category: "Van", Status: "Busy", Zone: "Near", Rating: "4", Expertise: "Airport Transfer", BookingDate: "2026-05-20", Phone: "094-787-9855" },
  { id: 10, Name: "JOHANN", Category: "Guide", Status: "Available", Zone: "Far", Rating: "5", Expertise: "Doi Inthanon", BookingDate: "2026-06-01", Phone: "086-3285939" },
  { id: 11, Name: "Mr. A", Category: "Car", Status: "Available", Zone: "Near", Rating: "3", Expertise: "City Tour", BookingDate: "", Phone: "081-1111111" },
  { id: 12, Name: "Blue Ocean", Category: "Boat", Status: "Available", Zone: "Far", Rating: "5", Expertise: "Island Hopping", BookingDate: "", Phone: "082-2222222" },
];

// ฟังก์ชันสร้างข้อมูลตัวอย่างเพิ่มเติมจนครบ 300 บรรทัด
const generateExtendedData = () => {
  const moreData = [];
  const statuses = ["Available", "Busy"];
  const zones = ["Near", "Far"];

  for (let i = 13; i <= 300; i++) {
    // คำนวณตัวเลขให้กลายเป็นตัวอักษร A-Z, AA-ZZ...
    let num = i - 12;
    let name = '';
    while (num > 0) {
      let rem = (num - 1) % 26;
      name = String.fromCharCode(65 + rem) + name;
      num = Math.floor((num - 1) / 26);
    }

    moreData.push({
      id: i,
      Name: name, // จะแสดงเป็น A, B, C...
      Category: "Guide",
      Status: statuses[i % 2],
      Zone: zones[i % 2],
      Rating: String((i % 5) + 1),
      Expertise: "General Tour",
      BookingDate: i % 3 === 0 ? "2026-06-15" : "",
      Phone: `080-000-${String(i).padStart(4, '0')}`
    });
  }
  return [...baseMockData, ...moreData];
};

const initialMockData = generateExtendedData();

export default function App() {
  const [data, setData] = useState(initialMockData);
  const [searchTerm, setSearchTerm] = useState('');
  const [errorMessage, setErrorMessage] = useState('');
  const [filters, setFilters] = useState({
    category: 'All',
    status: 'All',
    zone: 'All',
    rating: 'All'
  });
  
  // States for Inline Editing
  const [editingId, setEditingId] = useState(null);
  const [editFormData, setEditFormData] = useState({});

  const fileInputRef = useRef(null);

  // --- NATIVE CSV PARSER ---
  const handleFileUpload = (event) => {
    setErrorMessage('');
    const file = event.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onload = (e) => {
      const text = e.target.result;
      const lines = text.split(/\r?\n/).filter(line => line.trim() !== '');
      if (lines.length < 2) {
        setErrorMessage("CSV file seems empty or invalid format.");
        return;
      }

      // Extract headers
      const headers = lines[0].split(',').map(h => h.trim().replace(/^"|"$/g, ''));
      
      const parsedData = [];
      for (let i = 1; i < lines.length; i++) {
        // Handle commas inside quotes correctly
        const values = lines[i].split(/,(?=(?:(?:[^"]*"){2})*[^"]*$)/).map(v => v.replace(/^"|"$/g, '').trim());
        
        let rowObj = { id: Date.now() + i }; // Generate Unique ID for CRUD operations
        headers.forEach((header, index) => {
          rowObj[header] = values[index] || '';
        });
        
        // Ensure row has at least some valid identifier before pushing
        if (rowObj.Name || rowObj.Category) {
          parsedData.push(rowObj);
        }
      }
      
      setData(parsedData);
      
      if (fileInputRef.current) {
        fileInputRef.current.value = '';
      }
    };
    reader.readAsText(file);
  };

  // --- CRUD OPERATIONS ---
  const handleEditClick = (event, row) => {
    event.preventDefault();
    setEditingId(row.id);
    setEditFormData({ ...row });
  };

  const handleEditFormChange = (event) => {
    event.preventDefault();
    const fieldName = event.target.getAttribute("name");
    const fieldValue = event.target.value;
    setEditFormData(prev => ({
      ...prev,
      [fieldName]: fieldValue
    }));
  };

  const handleSaveClick = (event) => {
    event.preventDefault();
    const newData = [...data];
    const index = data.findIndex((item) => item.id === editingId);
    if (index !== -1) {
      newData[index] = editFormData;
      setData(newData);
    }
    setEditingId(null);
  };

  const handleCancelClick = () => {
    setEditingId(null);
  };

  const handleDeleteClick = (id) => {
    setData(data.filter(item => item.id !== id));
  };

  // --- KPI CALCULATIONS ---
  const stats = useMemo(() => {
    let guides = 0, cars = 0, boats = 0, available = 0;
    
    data.forEach(item => {
      const cat = item.Category?.toLowerCase() || '';
      if (cat === 'guide') guides++;
      if (cat === 'car' || cat === 'van') cars++;
      if (cat === 'boat') boats++;
      
      const status = item.Status?.toLowerCase() || '';
      if (status === 'available' || status === 'ว่าง') available++;
    });

    return { guides, cars, boats, available };
  }, [data]);

  // --- FILTERING LOGIC ---
  const filteredData = useMemo(() => {
    return data.filter(item => {
      // Search
      const matchesSearch = Object.values(item).some(val => 
        String(val).toLowerCase().includes(searchTerm.toLowerCase())
      );
      if (!matchesSearch) return false;

      // Category
      if (filters.category !== 'All') {
        const cat = item.Category?.toLowerCase() || '';
        if (filters.category === 'Car' && cat !== 'car' && cat !== 'van') return false;
        if (filters.category !== 'Car' && cat !== filters.category.toLowerCase()) return false;
      }

      // Status
      if (filters.status !== 'All') {
        const status = item.Status?.toLowerCase() || '';
        if (filters.status === 'Available' && status !== 'available' && status !== 'ว่าง') return false;
        if (filters.status === 'Busy' && status !== 'busy' && status !== 'ไม่ว่าง') return false;
      }

      // Zone
      if (filters.zone !== 'All') {
        const zone = item.Zone?.toLowerCase() || '';
        if (filters.zone === 'Near' && zone !== 'near' && zone !== 'ใกล้') return false;
        if (filters.zone === 'Far' && zone !== 'far' && zone !== 'ไกล') return false;
      }

      // Rating
      if (filters.rating !== 'All') {
        const rating = parseInt(item.Rating) || 0;
        if (rating < parseInt(filters.rating)) return false;
      }

      return true;
    });
  }, [data, filters, searchTerm]);

  // --- UI COMPONENTS ---
  const StatCard = ({ title, value, icon: Icon, colorClass }) => (
    <div className="bg-white rounded-xl shadow-sm p-6 border border-slate-100 flex items-center justify-between">
      <div>
        <p className="text-sm font-medium text-slate-500 mb-1">{title}</p>
        <h3 className="text-3xl font-bold text-slate-800">{value}</h3>
      </div>
      <div className={`p-4 rounded-full ${colorClass}`}>
        <Icon size={24} />
      </div>
    </div>
  );

  const getStatusBadge = (status) => {
    const isAvailable = String(status).toLowerCase() === 'available' || String(status).toLowerCase() === 'ว่าง';
    return isAvailable ? (
      <span className="inline-flex items-center px-2.5 py-1 rounded-full text-xs font-medium bg-emerald-100 text-emerald-800">
        <CheckCircle2 size={14} className="mr-1" /> Available
      </span>
    ) : (
      <span className="inline-flex items-center px-2.5 py-1 rounded-full text-xs font-medium bg-rose-100 text-rose-800">
        <XCircle size={14} className="mr-1" /> Busy
      </span>
    );
  };

  const getCategoryIcon = (category) => {
    const cat = String(category).toLowerCase();
    if (cat === 'guide') return <User size={16} className="text-blue-500" />;
    if (cat === 'boat') return <Ship size={16} className="text-cyan-500" />;
    if (cat === 'van') return <Bus size={16} className="text-indigo-500" />;
    return <Car size={16} className="text-indigo-500" />;
  };

  const RatingStars = ({ rating }) => {
    const num = parseInt(rating) || 0;
    return (
      <div className="flex">
        {[...Array(5)].map((_, i) => (
          <Star 
            key={i} 
            size={14} 
            className={i < num ? "fill-amber-400 text-amber-400" : "fill-slate-100 text-slate-200"} 
          />
        ))}
      </div>
    );
  };

  return (
    <div className="min-h-screen bg-slate-50 p-4 md:p-8 font-sans">
      <div className="max-w-7xl mx-auto space-y-6">
        
        {/* Header */}
        <header className="flex flex-col md:flex-row justify-between items-start md:items-center gap-4 bg-white p-6 rounded-xl shadow-sm border border-slate-100">
          <div>
            <h1 className="text-2xl font-bold text-slate-800">Tour Management Dashboard</h1>
            <p className="text-sm text-slate-500 mt-1">Manage Guides, Drivers, and Boats availability seamlessly.</p>
          </div>
          
          <div className="flex items-center gap-3">
            <input 
              type="file" 
              accept=".csv" 
              onChange={handleFileUpload} 
              ref={fileInputRef} 
              className="hidden" 
              id="csv-upload"
            />
            <label 
              htmlFor="csv-upload" 
              className="cursor-pointer inline-flex items-center justify-center px-4 py-2.5 bg-indigo-600 hover:bg-indigo-700 text-white text-sm font-medium rounded-lg transition-colors shadow-sm"
            >
              <Upload size={18} className="mr-2" />
              Upload CSV
            </label>
          </div>
        </header>

        {errorMessage && (
          <div className="bg-red-50 text-red-600 p-4 rounded-lg flex items-center text-sm border border-red-100">
            <AlertCircle size={16} className="mr-2" />
            {errorMessage}
          </div>
        )}

        {/* KPIs */}
        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          <StatCard title="Available Resources" value={stats.available} icon={CheckCircle2} colorClass="bg-emerald-100 text-emerald-600" />
          <StatCard title="Total Guides" value={stats.guides} icon={User} colorClass="bg-blue-100 text-blue-600" />
        </div>

        {/* Filters & Search */}
        <div className="bg-white p-4 rounded-xl shadow-sm border border-slate-100 flex flex-col md:flex-row gap-4 justify-between items-center">
          <div className="relative w-full md:w-64">
            <div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
              <Search size={16} className="text-slate-400" />
            </div>
            <input
              type="text"
              placeholder="Search anything..."
              className="pl-10 pr-4 py-2 w-full border border-slate-200 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:border-transparent"
              value={searchTerm}
              onChange={(e) => setSearchTerm(e.target.value)}
            />
          </div>

          <div className="flex flex-wrap items-center gap-3 w-full md:w-auto">
            <select 
              className="border border-slate-200 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-slate-50"
              value={filters.category}
              onChange={(e) => setFilters({...filters, category: e.target.value})}
            >
              <option value="All">All Categories</option>
              <option value="Guide">Guides</option>
              <option value="Car">Cars & Vans</option>
              <option value="Boat">Boats</option>
            </select>

            <select 
              className="border border-slate-200 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-slate-50"
              value={filters.status}
              onChange={(e) => setFilters({...filters, status: e.target.value})}
            >
              <option value="All">All Status</option>
              <option value="Available">Available</option>
              <option value="Busy">Busy</option>
            </select>

            <select 
              className="border border-slate-200 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-slate-50"
              value={filters.zone}
              onChange={(e) => setFilters({...filters, zone: e.target.value})}
            >
              <option value="All">All Zones</option>
              <option value="Near">Near</option>
              <option value="Far">Far</option>
            </select>

            <select 
              className="border border-slate-200 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-slate-50"
              value={filters.rating}
              onChange={(e) => setFilters({...filters, rating: e.target.value})}
            >
              <option value="All">Any Rating</option>
              <option value="4">4+ Stars</option>
              <option value="5">5 Stars</option>
            </select>
          </div>
        </div>

        {/* Data Table */}
        <div className="bg-white rounded-xl shadow-sm border border-slate-100 overflow-hidden">
          <div className="overflow-x-auto">
            <table className="w-full text-left border-collapse min-w-[900px]">
              <thead>
                <tr className="bg-slate-50 border-b border-slate-100 text-xs uppercase tracking-wider text-slate-500 font-semibold">
                  <th className="px-6 py-4 w-48">Name & Category</th>
                  <th className="px-6 py-4 w-32">Status</th>
                  <th className="px-6 py-4 w-24">Rating</th>
                  <th className="px-6 py-4 w-40">Expertise</th>
                  <th className="px-6 py-4 w-28">Zone</th>
                  <th className="px-6 py-4 w-48">Contact & Date</th>
                  <th className="px-6 py-4 w-24 text-center">Actions</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-slate-100">
                {filteredData.length > 0 ? (
                  filteredData.map((row) => (
                    editingId === row.id ? (
                      /* EDIT MODE ROW */
                      <tr key={row.id} className="bg-indigo-50/30">
                        <td className="px-6 py-3">
                          <input type="text" name="Name" value={editFormData.Name} onChange={handleEditFormChange} className="w-full mb-2 p-1.5 text-sm border rounded" placeholder="Name" />
                          <select name="Category" value={editFormData.Category} onChange={handleEditFormChange} className="w-full p-1.5 text-sm border rounded text-slate-600">
                            <option value="Guide">Guide</option>
                            <option value="Car">Car</option>
                            <option value="Van">Van</option>
                            <option value="Boat">Boat</option>
                          </select>
                        </td>
                        <td className="px-6 py-3">
                          <select name="Status" value={editFormData.Status} onChange={handleEditFormChange} className="w-full p-1.5 text-sm border rounded">
                            <option value="Available">Available (ว่าง)</option>
                            <option value="Busy">Busy (ไม่ว่าง)</option>
                          </select>
                        </td>
                        <td className="px-6 py-3">
                          <input type="number" min="1" max="5" name="Rating" value={editFormData.Rating} onChange={handleEditFormChange} className="w-full p-1.5 text-sm border rounded" />
                        </td>
                        <td className="px-6 py-3">
                          <input type="text" name="Expertise" value={editFormData.Expertise} onChange={handleEditFormChange} className="w-full p-1.5 text-sm border rounded" />
                        </td>
                        <td className="px-6 py-3">
                          <select name="Zone" value={editFormData.Zone} onChange={handleEditFormChange} className="w-full p-1.5 text-sm border rounded">
                            <option value="Near">Near (ใกล้)</option>
                            <option value="Far">Far (ไกล)</option>
                          </select>
                        </td>
                        <td className="px-6 py-3">
                          <input type="text" name="Phone" value={editFormData.Phone} onChange={handleEditFormChange} className="w-full mb-2 p-1.5 text-sm border rounded" placeholder="Phone" />
                          <input type="date" name="BookingDate" value={editFormData.BookingDate} onChange={handleEditFormChange} className="w-full p-1.5 text-sm border rounded" />
                        </td>
                        <td className="px-6 py-3">
                          <div className="flex items-center justify-center gap-2">
                            <button onClick={handleSaveClick} className="p-1.5 bg-emerald-100 text-emerald-600 rounded hover:bg-emerald-200 transition-colors" title="Save">
                              <Save size={16} />
                            </button>
                            <button onClick={handleCancelClick} className="p-1.5 bg-slate-200 text-slate-600 rounded hover:bg-slate-300 transition-colors" title="Cancel">
                              <X size={16} />
                            </button>
                          </div>
                        </td>
                      </tr>
                    ) : (
                      /* VIEW MODE ROW */
                      <tr key={row.id} className="hover:bg-slate-50 transition-colors">
                        <td className="px-6 py-4">
                          <div className="flex items-center">
                            <div className="flex-shrink-0 h-10 w-10 bg-slate-100 rounded-full flex items-center justify-center mr-3 border border-slate-200">
                              {getCategoryIcon(row.Category)}
                            </div>
                            <div>
                              <div className="font-semibold text-slate-800 text-sm">{row.Name || 'Unknown'}</div>
                              <div className="text-xs text-slate-500">{row.Category}</div>
                            </div>
                          </div>
                        </td>
                        <td className="px-6 py-4">
                          {getStatusBadge(row.Status)}
                        </td>
                        <td className="px-6 py-4">
                          <RatingStars rating={row.Rating} />
                        </td>
                        <td className="px-6 py-4">
                          <span className="text-sm text-slate-700 font-medium truncate block max-w-[150px]">{row.Expertise || '-'}</span>
                        </td>
                        <td className="px-6 py-4">
                          <div className="flex items-center text-sm text-slate-600">
                            <MapPin size={14} className="mr-1 text-slate-400" />
                            {row.Zone || '-'}
                          </div>
                        </td>
                        <td className="px-6 py-4">
                          <div className="text-sm text-slate-700 flex items-center mb-1">
                            <Phone size={14} className="mr-2 text-slate-400" />
                            {row.Phone || '-'}
                          </div>
                          {row.BookingDate && (
                            <div className="text-xs text-slate-500 flex items-center">
                              <Calendar size={14} className="mr-2 text-slate-400" />
                              {row.BookingDate}
                            </div>
                          )}
                        </td>
                        <td className="px-6 py-4">
                          <div className="flex items-center justify-center gap-2 opacity-0 group-hover:opacity-100 transition-opacity" style={{ opacity: 1 }}>
                            <button onClick={(e) => handleEditClick(e, row)} className="p-1.5 text-indigo-600 hover:bg-indigo-50 rounded transition-colors" title="Edit">
                              <Edit2 size={16} />
                            </button>
                            <button onClick={() => handleDeleteClick(row.id)} className="p-1.5 text-rose-500 hover:bg-rose-50 rounded transition-colors" title="Delete">
                              <Trash2 size={16} />
                            </button>
                          </div>
                        </td>
                      </tr>
                    )
                  ))
                ) : (
                  <tr>
                    <td colSpan="7" className="px-6 py-12 text-center text-slate-500">
                      <div className="flex flex-col items-center justify-center">
                        <Search size={32} className="text-slate-300 mb-3" />
                        <p className="text-sm">No records found matching your filters.</p>
                      </div>
                    </td>
                  </tr>
                )}
              </tbody>
            </table>
          </div>
          <div className="bg-slate-50 border-t border-slate-100 px-6 py-3 text-sm text-slate-500 flex justify-between items-center">
            <span>Showing <strong>{filteredData.length}</strong> entries</span>
            <span>Based on selected filters</span>
          </div>
        </div>

      </div>
    </div>
  );
}
