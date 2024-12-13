import React, { useState, useEffect } from 'react';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';
import { Alert, AlertDescription } from '@/components/ui/alert';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';

// Stock Certificate Management Component
const StockCertificateManagement = ({ userId, shareCount }) => {
  const [certificates, setCertificates] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  const [message, setMessage] = useState('');

  const generateCertificates = async () => {
    setIsLoading(true);
    try {
      const response = await fetch('/api/certificates/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ userId, shareCount })
      });
      
      const data = await response.json();
      setCertificates(data.certificates);
      setMessage('Certificates generated successfully');
    } catch (error) {
      setMessage('Error generating certificates');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="space-y-4">
      {certificates.length > 0 ? (
        <div className="space-y-2">
          {certificates.map((cert, index) => (
            <Alert key={cert.id}>
              <AlertDescription>
                Certificate #{index + 1}: {cert.certificateNumber}
              </AlertDescription>
            </Alert>
          ))}
        </div>
      ) : (
        <Button onClick={generateCertificates} disabled={isLoading}>
          {isLoading ? 'Generating...' : 'Generate Certificates'}
        </Button>
      )}
      
      {message && (
        <Alert>
          <AlertDescription>{message}</AlertDescription>
        </Alert>
      )}
    </div>
  );
};

// DIN Management Component
const DINManagement = ({ userId }) => {
  const [din, setDin] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [message, setMessage] = useState('');

  const generateDIN = async () => {
    setIsLoading(true);
    try {
      const response = await fetch('/api/din/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ userId })
      });
      
      const data = await response.json();
      setDin(data.din);
      setMessage('DIN generated successfully');
    } catch (error) {
      setMessage('Error generating DIN');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="space-y-4">
      {din ? (
        <Alert>
          <AlertDescription>DIN: {din}</AlertDescription>
        </Alert>
      ) : (
        <Button onClick={generateDIN} disabled={isLoading}>
          {isLoading ? 'Generating...' : 'Generate DIN'}
        </Button>
      )}
      
      {message && (
        <Alert>
          <AlertDescription>{message}</AlertDescription>
        </Alert>
      )}
    </div>
  );
};

// Registration Form Component
const RegistrationForm = ({ onRegistrationSuccess }) => {
  const [countries, setCountries] = useState([]);
  const [states, setStates] = useState([]);
  const [cities, setCities] = useState([]);
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [message, setMessage] = useState('');

  const [formData, setFormData] = useState({
    first_name: '',
    middle_name: '',
    last_name: '',
    phone_country_code: '',
    phone_number: '',
    email: '',
    street_address: '',
    apartment: '',
    address_line2: '',
    country_id: '',
    state_id: '',
    city_id: '',
    postal_code: ''
  });

  const [errors, setErrors] = useState({});

  useEffect(() => {
    const fetchCountries = async () => {
      try {
        const response = await fetch('/api/countries');
        const data = await response.json();
        setCountries(data);
      } catch (error) {
        setMessage('Error loading countries');
      }
    };
    fetchCountries();
  }, []);

  useEffect(() => {
    const fetchStates = async () => {
      if (formData.country_id) {
        try {
          const response = await fetch(`/api/states/${formData.country_id}`);
          const data = await response.json();
          setStates(data);
          setFormData(prev => ({ ...prev, state_id: '', city_id: '' }));
          setCities([]);
        } catch (error) {
          setMessage('Error loading states');
        }
      }
    };
    fetchStates();
  }, [formData.country_id]);

  useEffect(() => {
    const fetchCities = async () => {
      if (formData.state_id) {
        try {
          const response = await fetch(`/api/cities/${formData.state_id}`);
          const data = await response.json();
          setCities(data);
          setFormData(prev => ({ ...prev, city_id: '' }));
        } catch (error) {
          setMessage('Error loading cities');
        }
      }
    };
    fetchCities();
  }, [formData.state_id]);

  const validateForm = () => {
    const newErrors = {};
    if (!formData.first_name) newErrors.first_name = 'First name is required';
    if (!formData.last_name) newErrors.last_name = 'Last name is required';
    if (!formData.phone_number) newErrors.phone_number = 'Phone number is required';
    if (!formData.phone_country_code) newErrors.phone_country_code = 'Country code is required';
    if (!formData.street_address) newErrors.street_address = 'Street address is required';
    if (!formData.country_id) newErrors.country_id = 'Country is required';
    if (!formData.state_id) newErrors.state_id = 'State/Province is required';
    if (!formData.city_id) newErrors.city_id = 'City/Upazilla is required';

    const emailRegex = /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i;
    if (!formData.email) {
      newErrors.email = 'Email is required';
    } else if (!emailRegex.test(formData.email)) {
      newErrors.email = 'Invalid email address';
    }

    const phoneRegex = /^\d{1,15}$/;
    if (formData.phone_number && !phoneRegex.test(formData.phone_number)) {
      newErrors.phone_number = 'Invalid phone number';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: null }));
    }
  };

  const handleSelectChange = (name, value) => {
    setFormData(prev => ({ ...prev, [name]: value }));
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: null }));
    }
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!validateForm()) return;

    setIsSubmitting(true);
    try {
      const response = await fetch('/api/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData)
      });
      
      if (!response.ok) throw new Error('Registration failed');
      
      const result = await response.json();
      setMessage('Registration successful!');
      onRegistrationSuccess(result);
    } catch (error) {
      setMessage(error.message);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div className="grid grid-cols-2 gap-4">
        <div className="space-y-2">
          <Input
            name="first_name"
            value={formData.first_name}
            onChange={handleChange}
            placeholder="First Name"
            className={errors.first_name ? 'border-red-500' : ''}
          />
          {errors.first_name && (
            <span className="text-sm text-red-500">{errors.first_name}</span>
          )}
        </div>

        <Input
          name="middle_name"
          value={formData.middle_name}
          onChange={handleChange}
          placeholder="Middle Name (Optional)"
        />

        <div className="space-y-2">
          <Input
            name="last_name"
            value={formData.last_name}
            onChange={handleChange}
            placeholder="Last Name"
            className={errors.last_name ? 'border-red-500' : ''}
          />
          {errors.last_name && (
            <span className="text-sm text-red-500">{errors.last_name}</span>
          )}
        </div>
      </div>

      <div className="grid grid-cols-3 gap-4">
        <div className="space-y-2">
          <Select
            onValueChange={(value) => handleSelectChange('phone_country_code', value)}
            value={formData.phone_country_code}
          >
            <SelectTrigger className={errors.phone_country_code ? 'border-red-500' : ''}>
              <SelectValue placeholder="Country Code" />
            </SelectTrigger>
            <SelectContent>
              {countries.map(country => (
                <SelectItem key={country.id} value={country.phone_code}>
                  +{country.phone_code}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
          {errors.phone_country_code && (
            <span className="text-sm text-red-500">{errors.phone_country_code}</span>
          )}
        </div>

        <div className="col-span-2 space-y-2">
          <Input
            name="phone_number"
            value={formData.phone_number}
            onChange={handleChange}
            placeholder="Phone Number"
            className={errors.phone_number ? 'border-red-500' : ''}
          />
          {errors.phone_number && (
            <span className="text-sm text-red-500">{errors.phone_number}</span>
          )}
        </div>
      </div>

      <div className="space-y-2">
        <Input
          name="email"
          value={formData.email}
          onChange={handleChange}
          placeholder="Email Address"
          className={errors.email ? 'border-red-500' : ''}
        />
        {errors.email && (
          <span className="text-sm text-red-500">{errors.email}</span>
        )}
      </div>

      <div className="space-y-2">
        <Input
          name="street_address"
          value={formData.street_address}
          onChange={handleChange}
          placeholder="Street Address"
          className={errors.street_address ? 'border-red-500' : ''}
        />
        {errors.street_address && (
          <span className="text-sm text-red-500">{errors.street_address}</span>
        )}
      </div>

      <Input
        name="apartment"
        value={formData.apartment}
        onChange={handleChange}
        placeholder="Apartment (Optional)"
      />

      <Input
        name="address_line2"
        value={formData.address_line2}
        onChange={handleChange}
        placeholder="Address Line 2 (Optional)"
      />

      <div className="grid grid-cols-2 gap-4">
        <div className="space-y-2">
          <Select
            onValueChange={(value) => handleSelectChange('country_id', value)}
            value={formData.country_id}
          >
            <SelectTrigger className={errors.country_id ? 'border-red-500' : ''}>
              <SelectValue placeholder="Select Country" />
            </SelectTrigger>
            <SelectContent>
              {countries.map(country => (
                <SelectItem key={country.id} value={country.id}>
                  {country.name}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
          {errors.country_id && (
            <span className="text-sm text-red-500">{errors.country_id}</span>
          )}
        </div>

        <div className="space-y-2">
          <Select
            onValueChange={(value) => handleSelectChange('state_id', value)}
            value={formData.state_id}
            disabled={!states.length}
          >
            <SelectTrigger className={errors.state_id ? 'border-red-500' : ''}>
              <SelectValue placeholder="State/Province" />
            </SelectTrigger>
            <SelectContent>
              {states.map(state => (
                <SelectItem key={state.id} value={state.id}>
                  {state.name}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
          {errors.state_id && (
            <span className="text-sm text-red-500">{errors.state_id}</span>
          )}
        </div>

        <div className="space-y-2">
          <Select
            onValueChange={(value) => handleSelectChange('city_id', value)}
            value={formData.city_id}
            disabled={!cities.length}
          >
            <SelectTrigger className={errors.city_id ? 'border-red-500' : ''}>
              <SelectValue placeholder="City/Upazilla" />
            </SelectTrigger>
            <SelectContent>
              {cities.map(city => (
                <SelectItem key={city.id} value={city.id}>
                  {city.name}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
          {errors.city_id && (
            <span className="text-sm text-red-500">{errors.city_id}</span>
          )}
        </div>

        <Input
          name="postal_code"
          value={formData.postal_code}
          onChange={handleChange}
          placeholder="Postal Code"
        />
      </div>

      <Button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Registering...' : 'Register'}
      </Button>

      {message && (
        <Alert>
          <AlertDescription>{message}</AlertDescription>
        </Alert>
      )}
    </form>
  );
};

// Main Dashboard Component
const ManagementDashboard = () => {
  const [activeUser, setActiveUser] = useState(null);
  const [paymentStatus, setPaymentStatus] = useState(null);
  const [shareCount, setShareCount] = useState(1);
  const [paymentAmount, setPaymentAmount] = useState(100);

  const handlePayment = async () => {
    try {
      const response = await fetch('/api/process-payment', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          userId: activeUser.id,
          amount: paymentAmount,
          shares: shareCount
        })
      });
      
      if (!response.ok) throw new Error('Payment failed');
      
      const result = await response.json();
      setPaymentStatus('success');
    } catch (error) {
      setPaymentStatus('failed');
    }
  };

  const handleShareCountChange = (value) => {
    const count = parseInt(value);
    setShareCount(count);
    setPaymentAmount(count * 100);
  };

  return (
    <div className="container mx-auto p-4">
      <Tabs defaultValue="registration" className="w-full">
        <TabsList className="grid w-full grid-cols-2">
          <TabsTrigger value="registration">User Registration</TabsTrigger>
          <TabsTrigger 
            value="management"
            disabled={!activeUser || paymentStatus !== 'success'}
          >
            DIN & Stock Management
          </TabsTrigger>
        </TabsList>

        <TabsContent value="registration">
          <Card>
            <CardContent className="space-y-4">
              <RegistrationForm 
                onRegistrationSuccess={(user) => setActiveUser(user)} 
              />
              
              {activeUser && paymentStatus !== 'success' && (
                <div className="mt-6 space-y-4 border-t pt-4">
                  <h3 className="text-lg font-semibold">Share Purchase</h3>
                  <div className="grid grid-cols-2 gap-4">
                    <div className="space-y-2">
                      <label className="text-sm font-medium">Number of Shares</label>
                      <Select
                        value={shareCount.toString()}
                        onValueChange={handleShareCountChange}
                      >
                        <SelectTrigger>
                          <SelectValue placeholder="Select shares" />
                        </SelectTrigger>
                        <SelectContent>
                          {[...Array(10)].map((_, i) => (
                            <SelectItem key={i + 1} value={(i + 1).toString()}>
                              {i + 1} {i === 0 ? 'Share' : 'Shares'}
                            </SelectItem>
                          ))}
                        </SelectContent>
                      </Select>
                    </div>
                    
                    <div className="space-y-2">
                      <label className="text-sm font-medium">Total Amount</label>
                      <Input
                        value={`$${paymentAmount.toFixed(2)}`}
                        disabled
                        className="bg-gray-50"
                      />
                    </div>
                  </div>

                  <Button 
                    onClick={handlePayment}
                    className="w-full"
                  >
                    Process Payment
                  </Button>

                  {paymentStatus === 'failed' && (
                    <Alert variant="destructive">
                      <AlertDescription>
                        Payment processing failed. Please try again.
                      </AlertDescription>
                    </Alert>
                  )}
                </div>
              )}

              {paymentStatus === 'success' && (
                <Alert>
                  <AlertDescription>
                    Payment successful! You can now proceed to DIN & Stock Management.
                  </AlertDescription>
                </Alert>
              )}
            </CardContent>
          </Card>
        </TabsContent>

        <TabsContent value="management">
          <Card>
            <CardContent className="space-y-4">
              <div className="grid gap-4">
                <div className="bg-gray-50 p-4 rounded-lg">
                  <h3 className="font-medium mb-2">Registered User</h3>
                  <p>Name: {activeUser?.first_name} {activeUser?.last_name}</p>
                  <p>Email: {activeUser?.email}</p>
                  <p>Shares Purchased: {shareCount}</p>
                </div>

                <div className="border-t pt-4">
                  <h3 className="font-medium mb-4">DIN Management</h3>
                  <DINManagement userId={activeUser?.id} />
                </div>

                <div className="border-t pt-4">
                  <h3 className="font-medium mb-4">Stock Certificates</h3>
                  <StockCertificateManagement 
                    userId={activeUser?.id}
                    shareCount={shareCount}
                  />
                </div>
              </div>
            </CardContent>
          </Card>
        </TabsContent>
      </Tabs>
    </div>
  );
};

export default ManagementDashboard;
