I'll modify the dashboard to work with mock data so it can be demonstrated without an API.



```tsx
import React, { useState, useEffect } from 'react';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Card, CardContent } from '@/components/ui/card';
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';
import { Alert, AlertDescription } from '@/components/ui/alert';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';

// Mock Data
const MOCK_COUNTRIES = [
  { id: '1', name: 'United States', phone_code: '1' },
  { id: '2', name: 'United Kingdom', phone_code: '44' },
  { id: '3', name: 'Canada', phone_code: '1' },
  { id: '4', name: 'Australia', phone_code: '61' }
];

const MOCK_STATES = {
  '1': [
    { id: '1', name: 'California' },
    { id: '2', name: 'New York' },
    { id: '3', name: 'Texas' }
  ],
  '2': [
    { id: '4', name: 'England' },
    { id: '5', name: 'Scotland' },
    { id: '6', name: 'Wales' }
  ]
};

const MOCK_CITIES = {
  '1': [
    { id: '1', name: 'Los Angeles' },
    { id: '2', name: 'San Francisco' },
    { id: '3', name: 'San Diego' }
  ],
  '2': [
    { id: '4', name: 'New York City' },
    { id: '5', name: 'Buffalo' },
    { id: '6', name: 'Albany' }
  ]
};

const DINManagement = ({ userId }) => {
  const [din, setDin] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [message, setMessage] = useState('');

  const generateDIN = async () => {
    setIsLoading(true);
    // Simulate API call
    setTimeout(() => {
      const mockDIN = `DIN${Math.random().toString(36).substr(2, 9).toUpperCase()}`;
      setDin(mockDIN);
      setMessage('DIN generated successfully');
      setIsLoading(false);
    }, 1000);
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

const StockCertificateManagement = ({ userId, shareCount }) => {
  const [certificates, setCertificates] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  const [message, setMessage] = useState('');

  const generateCertificates = async () => {
    setIsLoading(true);
    // Simulate API call
    setTimeout(() => {
      const mockCertificates = Array(shareCount).fill(null).map((_, index) => ({
        id: index + 1,
        certificateNumber: `CERT-${Math.random().toString(36).substr(2, 6).toUpperCase()}`
      }));
      setCertificates(mockCertificates);
      setMessage('Certificates generated successfully');
      setIsLoading(false);
    }, 1500);
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

const RegistrationForm = ({ onRegistrationSuccess }) => {
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

  const [states, setStates] = useState([]);
  const [cities, setCities] = useState([]);
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [message, setMessage] = useState('');
  const [errors, setErrors] = useState({});

  // Load states when country changes
  useEffect(() => {
    if (formData.country_id) {
      setStates(MOCK_STATES[formData.country_id] || []);
      setFormData(prev => ({ ...prev, state_id: '', city_id: '' }));
      setCities([]);
    }
  }, [formData.country_id]);

  // Load cities when state changes
  useEffect(() => {
    if (formData.state_id) {
      setCities(MOCK_CITIES[formData.state_id] || []);
      setFormData(prev => ({ ...prev, city_id: '' }));
    }
  }, [formData.state_id]);

  const validateForm = () => {
    const newErrors = {};
    if (!formData.first_name) newErrors.first_name = 'First name is required';
    if (!formData.last_name) newErrors.last_name = 'Last name is required';
    if (!formData.email) newErrors.email = 'Email is required';
    if (!formData.phone_number) newErrors.phone_number = 'Phone number is required';
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!validateForm()) return;
    
    setIsSubmitting(true);
    // Simulate API call
    setTimeout(() => {
      const mockUser = {
        id: Math.random().toString(36).substr(2, 9),
        ...formData
      };
      onRegistrationSuccess(mockUser);
      setMessage('Registration successful!');
      setIsSubmitting(false);
    }, 1000);
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-6">
      <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div>
          <Input
            placeholder="First Name"
            value={formData.first_name}
            onChange={e => setFormData(prev => ({ ...prev, first_name: e.target.value }))}
            className={errors.first_name ? 'border-red-500' : ''}
          />
          {errors.first_name && (
            <span className="text-sm text-red-500">{errors.first_name}</span>
          )}
        </div>
        
        <Input
          placeholder="Middle Name (Optional)"
          value={formData.middle_name}
          onChange={e => setFormData(prev => ({ ...prev, middle_name: e.target.value }))}
        />
        
        <div>
          <Input
            placeholder="Last Name"
            value={formData.last_name}
            onChange={e => setFormData(prev => ({ ...prev, last_name: e.target.value }))}
            className={errors.last_name ? 'border-red-500' : ''}
          />
          {errors.last_name && (
            <span className="text-sm text-red-500">{errors.last_name}</span>
          )}
        </div>
      </div>

      <div className="space-y-4">
        <div className="grid grid-cols-3 gap-4">
          <Select
            value={formData.phone_country_code}
            onValueChange={value => setFormData(prev => ({ ...prev, phone_country_code: value }))}
          >
            <SelectTrigger>
              <SelectValue placeholder="Code" />
            </SelectTrigger>
            <SelectContent>
              {MOCK_COUNTRIES.map(country => (
                <SelectItem key={country.id} value={country.phone_code}>
                  +{country.phone_code}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>

          <div className="col-span-2">
            <Input
              placeholder="Phone Number"
              value={formData.phone_number}
              onChange={e => setFormData(prev => ({ ...prev, phone_number: e.target.value }))}
              className={errors.phone_number ? 'border-red-500' : ''}
            />
            {errors.phone_number && (
              <span className="text-sm text-red-500">{errors.phone_number}</span>
            )}
          </div>
        </div>

        <div>
          <Input
            placeholder="Email Address"
            type="email"
            value={formData.email}
            onChange={e => setFormData(prev => ({ ...prev, email: e.target.value }))}
            className={errors.email ? 'border-red-500' : ''}
          />
          {errors.email && (
            <span className="text-sm text-red-500">{errors.email}</span>
          )}
        </div>

        <Input
          placeholder="Street Address"
          value={formData.street_address}
          onChange={e => setFormData(prev => ({ ...prev, street_address: e.target.value }))}
        />

        <Input
          placeholder="Apartment (Optional)"
          value={formData.apartment}
          onChange={e => setFormData(prev => ({ ...prev, apartment: e.target.value }))}
        />

        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          <Select
            value={formData.country_id}
            onValueChange={value => setFormData(prev => ({ ...prev, country_id: value }))}
          >
            <SelectTrigger>
              <SelectValue placeholder="Select Country" />
            </SelectTrigger>
            <SelectContent>
              {MOCK_COUNTRIES.map(country => (
                <SelectItem key={country.id} value={country.id}>{country.name}</SelectItem>
              ))}
            </SelectContent>
          </Select>

          <Select
            value={formData.state_id}
            onValueChange={value => setFormData(prev => ({ ...prev, state_id: value }))}
            disabled={!states.length}
          >
            <SelectTrigger>
              <SelectValue placeholder="Select State" />
            </SelectTrigger>
            <SelectContent>
              {states.map(state => (
                <SelectItem key={state.id} value={state.id}>{state.name}</SelectItem>
              ))}
            </SelectContent>
          </Select>

          <Select
            value={formData.city_id}
            onValueChange={value => setFormData(prev => ({ ...prev, city_id: value }))}
            disabled={!cities.length}
          >
            <SelectTrigger>
              <SelectValue placeholder="Select City" />
            </SelectTrigger>
            <SelectContent>
              {cities.map(city => (
                <SelectItem key={city.id} value={city.id}>{city.name}</SelectItem>
              ))}
            </SelectContent>
          </Select>

          <Input
            placeholder="Postal Code"
            value={formData.postal_code}
            onChange={e => setFormData(prev => ({ ...prev, postal_code: e.target.value }))}
          />
        </div>
      </div>

      <Button type="submit" className="w-full" disabled={isSubmitting}>
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

const ManagementDashboard = () => {
  const [activeUser, setActiveUser] = useState(null);
  const [paymentStatus, setPaymentStatus] = useState(null);
  const [shareCount, setShareCount] = useState(1);
  const [paymentAmount, setPaymentAmount] = useState(100);

  const handlePayment = async () => {
    // Simulate payment processing
    setTimeout(() => {
      setPaymentStatus('success');
    }, 1000);
  };

  return (
    <div className="container mx-auto p-4">
      <Tabs defaultValue="registration" className="w-full">
        <TabsList className="grid w-full grid-cols-2">
          <TabsTrigger value="registration">User Registration</TabsTrigger>
          <TabsTrigger value="management" disabled={!activeUser || paymentStatus !== 'success'}>
            DIN & Stock Management
          </TabsTrigger>
        </TabsList>

        <TabsContent value="registration">
          <Card>
            <CardContent className="pt-6">
              <RegistrationForm onRegistrationSuccess={setActiveUser} />
              
              {activeUser && paymentStatus !== 'success' && (
                <div className="mt-6 space-y-4 border-t pt-4">
                  <h3 className="text-lg font-medium">Share Purchase</h3>
                  <div className="grid grid-cols-2 gap-4">
                    <Select
                      value={shareCount.toString()}
                      onValueChange={(value) => {
                        const count = parseInt(value);
                        setShareCount(count);
                        setPaymentAmount(count * 100);
                      }}
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

                    <Input
                      value={`$${paymentAmount.toFixed(2)}`}
                      disabled
                      className="bg-gray-50"
                    />
                  </div>

                  <Button onClick={handlePayment} className="w-full">
                    Process Payment
                  </Button>
                </div>
              )}

              {paymentStatus === 'success' && (
                <Alert className="mt-4">
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
            <CardContent className="space-y-6 pt-6">
              <div className="bg-gray-50 p-4 rounded-lg">
                <h3 className="font-medium mb-2">User Details</h3>
                <div className="space-y-1">
                  <p className="text-sm">
                    Name: {activeUser?.first_name} {activeUser?.middle_name ? `${activeUser.middle_name} ` : ''}{activeUser?.last_name}
                  </p>
                  <p className="text-sm">Email: {activeUser?.email}</p>
                  <p className="text-sm">Phone: +{activeUser?.phone_country_code} {activeUser?.phone_number}</p>
                  <p className="text-sm">Shares Purchased: {shareCount}</p>
                  <p className="text-sm">Total Investment: ${paymentAmount.toFixed(2)}</p>
                </div>
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

              <div className="border-t pt-4">
                <h3 className="font-medium mb-4">Activity Timeline</h3>
                <div className="space-y-2">
                  {[
                    { date: new Date().toLocaleDateString(), action: 'Registration completed' },
                    { date: new Date().toLocaleDateString(), action: `Payment processed for ${shareCount} shares` },
                  ].map((activity, index) => (
                    <div key={index} className="flex items-center justify-between p-2 bg-gray-50 rounded">
                      <span className="text-sm">{activity.action}</span>
                      <span className="text-xs text-gray-500">{activity.date}</span>
                    </div>
                  ))}
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

```


