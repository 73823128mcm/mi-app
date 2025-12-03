import React, { useState, useEffect } from 'react';
import { Plus, Edit2, Trash2, Search, Calendar, Heart, Activity } from 'lucide-react';

// Simulación de base de datos con almacenamiento local
const PetCareApp = () => {
  const [pets, setPets] = useState([]);
  const [appointments, setAppointments] = useState([]);
  const [showAddPet, setShowAddPet] = useState(false);
  const [showAddAppointment, setShowAddAppointment] = useState(false);
  const [searchTerm, setSearchTerm] = useState('');
  const [editingPet, setEditingPet] = useState(null);
  const [formData, setFormData] = useState({
    name: '',
    species: '',
    breed: '',
    age: '',
    weight: '',
    owner: ''
  });
  const [appointmentData, setAppointmentData] = useState({
    petId: '',
    date: '',
    time: '',
    type: '',
    notes: ''
  });
  const [errors, setErrors] = useState({});

  // Cargar datos al iniciar
  useEffect(() => {
    const savedPets = localStorage.getItem('pets');
    const savedAppointments = localStorage.getItem('appointments');
    if (savedPets) setPets(JSON.parse(savedPets));
    if (savedAppointments) setAppointments(JSON.parse(savedAppointments));
  }, []);

  // Guardar datos cuando cambien
  useEffect(() => {
    localStorage.setItem('pets', JSON.stringify(pets));
  }, [pets]);

  useEffect(() => {
    localStorage.setItem('appointments', JSON.stringify(appointments));
  }, [appointments]);

  // Validación de formulario
  const validatePetForm = () => {
    const newErrors = {};
    
    if (!formData.name.trim()) {
      newErrors.name = 'El nombre es requerido';
    }
    
    if (!formData.species.trim()) {
      newErrors.species = 'La especie es requerida';
    }
    
    if (formData.age && (isNaN(formData.age) || formData.age < 0)) {
      newErrors.age = 'La edad debe ser un número positivo';
    }
    
    if (formData.weight && (isNaN(formData.weight) || formData.weight <= 0)) {
      newErrors.weight = 'El peso debe ser un número positivo';
    }
    
    if (!formData.owner.trim()) {
      newErrors.owner = 'El nombre del dueño es requerido';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const validateAppointmentForm = () => {
    const newErrors = {};
    
    if (!appointmentData.petId) {
      newErrors.petId = 'Debe seleccionar una mascota';
    }
    
    if (!appointmentData.date) {
      newErrors.date = 'La fecha es requerida';
    }
    
    if (!appointmentData.time) {
      newErrors.time = 'La hora es requerida';
    }
    
    if (!appointmentData.type) {
      newErrors.type = 'El tipo de cita es requerido';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  // Función para insertar/actualizar mascota (CRUD)
  const handleSavePet = () => {
    if (!validatePetForm()) return;

    if (editingPet) {
      // Actualizar mascota existente
      setPets(pets.map(pet => 
        pet.id === editingPet.id 
          ? { ...formData, id: editingPet.id }
          : pet
      ));
      setEditingPet(null);
    } else {
      // Insertar nueva mascota
      const newPet = {
        ...formData,
        id: Date.now(),
        createdAt: new Date().toISOString()
      };
      setPets([...pets, newPet]);
    }

    // Resetear formulario
    setFormData({
      name: '',
      species: '',
      breed: '',
      age: '',
      weight: '',
      owner: ''
    });
    setShowAddPet(false);
    setErrors({});
  };

  // Función para eliminar mascota
  const handleDeletePet = (id) => {
    if (window.confirm('¿Está seguro de eliminar esta mascota?')) {
      setPets(pets.filter(pet => pet.id !== id));
      // Eliminar citas asociadas
      setAppointments(appointments.filter(apt => apt.petId !== id));
    }
  };

  // Función para editar mascota
  const handleEditPet = (pet) => {
    setFormData(pet);
    setEditingPet(pet);
    setShowAddPet(true);
  };

  // Función para agregar cita
  const handleSaveAppointment = () => {
    if (!validateAppointmentForm()) return;

    const newAppointment = {
      ...appointmentData,
      id: Date.now(),
      createdAt: new Date().toISOString()
    };
    
    setAppointments([...appointments, newAppointment]);
    setAppointmentData({
      petId: '',
      date: '',
      time: '',
      type: '',
      notes: ''
    });
    setShowAddAppointment(false);
    setErrors({});
  };

  // Algoritmo de búsqueda optimizado
  const filteredPets = pets.filter(pet => {
    const search = searchTerm.toLowerCase();
    return (
      pet.name.toLowerCase().includes(search) ||
      pet.species.toLowerCase().includes(search) ||
      pet.breed?.toLowerCase().includes(search) ||
      pet.owner.toLowerCase().includes(search)
    );
  });

  // Obtener nombre de mascota por ID
  const getPetName = (petId) => {
    const pet = pets.find(p => p.id === parseInt(petId));
    return pet ? pet.name : 'Mascota desconocida';
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 p-4">
      <div className="max-w-6xl mx-auto">
        {/* Header */}
        <div className="bg-white rounded-lg shadow-lg p-6 mb-6">
          <div className="flex items-center justify-between flex-wrap gap-4">
            <div className="flex items-center gap-3">
              <Heart className="text-pink-500" size={32} />
              <h1 className="text-3xl font-bold text-gray-800">Cuidado de Animales</h1>
            </div>
            <div className="flex gap-2">
              <button
                onClick={() => {
                  setShowAddPet(true);
                  setEditingPet(null);
                  setFormData({
                    name: '',
                    species: '',
                    breed: '',
                    age: '',
                    weight: '',
                    owner: ''
                  });
                }}
                className="bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-lg flex items-center gap-2 transition"
              >
                <Plus size={20} />
                <span className="hidden sm:inline">Nueva Mascota</span>
              </button>
              <button
                onClick={() => setShowAddAppointment(true)}
                className="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg flex items-center gap-2 transition"
              >
                <Calendar size={20} />
                <span className="hidden sm:inline">Nueva Cita</span>
              </button>
            </div>
          </div>

          {/* Barra de búsqueda */}
          <div className="mt-4 relative">
            <Search className="absolute left-3 top-3 text-gray-400" size={20} />
            <input
              type="text"
              placeholder="Buscar por nombre, especie, raza o dueño..."
              value={searchTerm}
              onChange={(e) => setSearchTerm(e.target.value)}
              className="w-full pl-10 pr-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-transparent"
            />
          </div>
        </div>

        {/* Modal para agregar/editar mascota */}
        {showAddPet && (
          <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
            <div className="bg-white rounded-lg shadow-xl p-6 w-full max-w-md max-h-[90vh] overflow-y-auto">
              <h2 className="text-2xl font-bold mb-4">
                {editingPet ? 'Editar Mascota' : 'Nueva Mascota'}
              </h2>
              
              <div className="space-y-4">
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Nombre *
                  </label>
                  <input
                    type="text"
                    value={formData.name}
                    onChange={(e) => setFormData({...formData, name: e.target.value})}
                    className={`w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-indigo-500 ${errors.name ? 'border-red-500' : 'border-gray-300'}`}
                  />
                  {errors.name && <p className="text-red-500 text-sm mt-1">{errors.name}</p>}
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Especie *
                  </label>
                  <select
                    value={formData.species}
                    onChange={(e) => setFormData({...formData, species: e.target.value})}
                    className={`w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-indigo-500 ${errors.species ? 'border-red-500' : 'border-gray-300'}`}
                  >
                    <option value="">Seleccionar...</option>
                    <option value="Perro">Perro</option>
                    <option value="Gato">Gato</option>
                    <option value="Ave">Ave</option>
                    <option value="Conejo">Conejo</option>
                    <option value="Otro">Otro</option>
                  </select>
                  {errors.species && <p className="text-red-500 text-sm mt-1">{errors.species}</p>}
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Raza
                  </label>
                  <input
                    type="text"
                    value={formData.breed}
                    onChange={(e) => setFormData({...formData, breed: e.target.value})}
                    className="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
                  />
                </div>

                <div className="grid grid-cols-2 gap-4">
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">
                      Edad (años)
                    </label>
                    <input
                      type="number"
                      value={formData.age}
                      onChange={(e) => setFormData({...formData, age: e.target.value})}
                      className={`w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-indigo-500 ${errors.age ? 'border-red-500' : 'border-gray-300'}`}
                    />
                    {errors.age && <p className="text-red-500 text-sm mt-1">{errors.age}</p>}
                  </div>

                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">
                      Peso (kg)
                    </label>
                    <input
                      type="number"
                      step="0.1"
                      value={formData.weight}
                      onChange={(e) => setFormData({...formData, weight: e.target.value})}
                      className={`w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-indigo-500 ${errors.weight ? 'border-red-500' : 'border-gray-300'}`}
                    />
                    {errors.weight && <p className="text-red-500 text-sm mt-1">{errors.weight}</p>}
                  </div>
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Dueño *
                  </label>
                  <input
                    type="text"
                    value={formData.owner}
                    onChange={(e) => setFormData({...formData, owner: e.target.value})}
                    className={`w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-indigo-500 ${errors.owner ? 'border-red-500' : 'border-gray-300'}`}
                  />
                  {errors.owner && <p className="text-red-500 text-sm mt-1">{errors.owner}</p>}
                </div>
              </div>

              <div className="flex gap-2 mt-6">
                <button
                  onClick={handleSavePet}
                  className="flex-1 bg-indigo-600 hover:bg-indigo-700 text-white px-4 py-2 rounded-lg transition"
                >
                  {editingPet ? 'Actualizar' : 'Guardar'}
                </button>
                <button
                  onClick={() => {
                    setShowAddPet(false);
                    setEditingPet(null);
                    setErrors({});
                  }}
                  className="flex-1 bg-gray-300 hover:bg-gray-400 text-gray-800 px-4 py-2 rounded-lg transition"
                >
                  Cancelar
                </button>
              </div>
            </div>
          </div>
        )}

        {/* Modal para agregar cita */}
        {showAddAppointment && (
          <div className="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50">
            <div className="bg-white rounded-lg shadow-xl p-6 w-full max-w-md max-h-[90vh] overflow-y-auto">
              <h2 className="text-2xl font-bold mb-4">Nueva Cita Veterinaria</h2>
              
              <div className="space-y-4">
                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Mascota *
                  </label>
                  <select
                    value={appointmentData.petId}
                    onChange={(e) => setAppointmentData({...appointmentData, petId: e.target.value})}
                    className={`w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-indigo-500 ${errors.petId ? 'border-red-500' : 'border-gray-300'}`}
                  >
                    <option value="">Seleccionar mascota...</option>
                    {pets.map(pet => (
                      <option key={pet.id} value={pet.id}>
                        {pet.name} - {pet.species}
                      </option>
                    ))}
                  </select>
                  {errors.petId && <p className="text-red-500 text-sm mt-1">{errors.petId}</p>}
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Fecha *
                  </label>
                  <input
                    type="date"
                    value={appointmentData.date}
                    onChange={(e) => setAppointmentData({...appointmentData, date: e.target.value})}
                    className={`w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-indigo-500 ${errors.date ? 'border-red-500' : 'border-gray-300'}`}
                  />
                  {errors.date && <p className="text-red-500 text-sm mt-1">{errors.date}</p>}
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Hora *
                  </label>
                  <input
                    type="time"
                    value={appointmentData.time}
                    onChange={(e) => setAppointmentData({...appointmentData, time: e.target.value})}
                    className={`w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-indigo-500 ${errors.time ? 'border-red-500' : 'border-gray-300'}`}
                  />
                  {errors.time && <p className="text-red-500 text-sm mt-1">{errors.time}</p>}
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Tipo de Cita *
                  </label>
                  <select
                    value={appointmentData.type}
                    onChange={(e) => setAppointmentData({...appointmentData, type: e.target.value})}
                    className={`w-full px-3 py-2 border rounded-lg focus:ring-2 focus:ring-indigo-500 ${errors.type ? 'border-red-500' : 'border-gray-300'}`}
                  >
                    <option value="">Seleccionar...</option>
                    <option value="Consulta General">Consulta General</option>
                    <option value="Vacunación">Vacunación</option>
                    <option value="Emergencia">Emergencia</option>
                    <option value="Control">Control</option>
                    <option value="Cirugía">Cirugía</option>
                  </select>
                  {errors.type && <p className="text-red-500 text-sm mt-1">{errors.type}</p>}
                </div>

                <div>
                  <label className="block text-sm font-medium text-gray-700 mb-1">
                    Notas
                  </label>
                  <textarea
                    value={appointmentData.notes}
                    onChange={(e) => setAppointmentData({...appointmentData, notes: e.target.value})}
                    rows="3"
                    className="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
                  />
                </div>
              </div>

              <div className="flex gap-2 mt-6">
                <button
                  onClick={handleSaveAppointment}
                  className="flex-1 bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg transition"
                >
                  Guardar
                </button>
                <button
                  onClick={() => {
                    setShowAddAppointment(false);
                    setErrors({});
                  }}
                  className="flex-1 bg-gray-300 hover:bg-gray-400 text-gray-800 px-4 py-2 rounded-lg transition"
                >
                  Cancelar
                </button>
              </div>
            </div>
          </div>
        )}

        {/* Lista de mascotas */}
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 mb-6">
          {filteredPets.length === 0 ? (
            <div className="col-span-full bg-white rounded-lg shadow p-8 text-center">
              <Activity className="mx-auto text-gray-400 mb-4" size={48} />
              <p className="text-gray-600">
                {searchTerm ? 'No se encontraron mascotas' : 'No hay mascotas registradas'}
              </p>
            </div>
          ) : (
            filteredPets.map(pet => (
              <div key={pet.id} className="bg-white rounded-lg shadow-lg p-6 hover:shadow-xl transition">
                <div className="flex justify-between items-start mb-4">
                  <div>
                    <h3 className="text-xl font-bold text-gray-800">{pet.name}</h3>
                    <p className="text-gray-600">{pet.species}</p>
                  </div>
                  <div className="flex gap-2">
                    <button
                      onClick={() => handleEditPet(pet)}
                      className="text-blue-600 hover:text-blue-800 transition"
                    >
                      <Edit2 size={18} />
                    </button>
                    <button
                      onClick={() => handleDeletePet(pet.id)}
                      className="text-red-600 hover:text-red-800 transition"
                    >
                      <Trash2 size={18} />
                    </button>
                  </div>
                </div>
                
                <div className="space-y-2 text-sm">
                  {pet.breed && <p><strong>Raza:</strong> {pet.breed}</p>}
                  {pet.age && <p><strong>Edad:</strong> {pet.age} años</p>}
                  {pet.weight && <p><strong>Peso:</strong> {pet.weight} kg</p>}
                  <p><strong>Dueño:</strong> {pet.owner}</p>
                </div>
              </div>
            ))
          )}
        </div>

        {/* Lista de citas */}
        {appointments.length > 0 && (
          <div className="bg-white rounded-lg shadow-lg p-6">
            <h2 className="text-2xl font-bold mb-4 flex items-center gap-2">
              <Calendar className="text-green-600" />
              Próximas Citas
