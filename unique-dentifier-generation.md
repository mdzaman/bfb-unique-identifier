I'll enhance the dashboard with proper share price calculation, payment processing, and demo data functionality.



```tsx
import React, { useState, useEffect } from 'react';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Card, CardContent } from '@/components/ui/card';
import { Input } from '@/components/ui/input';
import { Button } from '@/components/ui/button';
import { Alert, AlertDescription } from '@/components/ui/alert';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';

// Constants
const SHARE_PRICE = 100; // $100 per share
const MAX_SHARES = 10;

const USA_STATES = {
  "CA": {
    name: "California",
    cities: ["Los Angeles", "San Francisco", "San Diego", "Sacramento"],
    zipCodePattern: "^9[0-6]\\d{3}$"
  },
  "NY": {
    name: "New York",
    cities: ["New York City", "Buffalo", "Albany", "Rochester"],
    zipCodePattern: "^1[0-4]\\d{3}$"
  },
  "TX": {
    name: "Texas",
    cities: ["Houston", "Austin", "Dallas", "San Antonio"],
    zipCodePattern: "^7[5-9]\\d{3}$"
  }
};

const PaymentSection = ({ amount, onSuccess }) => {
  const [cardNumber, setCardNumber] = useState('');
  const [expiry, setExpiry] = useState('');
  const [cvv, setCvv] = useState('');
  const [processing, setProcessing] = useState(false);
  const [error, setError] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    setError('');
    setProcessing(true);

    // Validate card details
    if (!cardNumber.match(/^\d{16}$/)) {
      setError('Invalid card number');
      setProcessing(false);
      return;
    }
    if (!expiry.match(/^\d{2}\/\d{2}$/)) {
      setError('Invalid expiry date');
      setProcessing(false);
      return;
    }
    if (!cvv.match(/^\d{3}$/)) {
      setError('Invalid CVV');
      setProcessing(false);
      return;
    }

    // Simulate payment processing
    setTimeout(() => {
      setProcessing(false);
      onSuccess();
    }, 2000);
  };

  return (
    <div className="space-y-4">
      <div className="bg-blue-50 p-4 rounded-lg mb-4">
        <p className="text-lg font-medium">Total Amount: ${amount.toFixed(2)}</p>
      </div>
      <form onSubmit={handleSubmit} className="space-y-4">
        <div>
          <Input
            placeholder="Card Number"
            value={cardNumber}
            onChange={(e) => setCardNumber(e.target.value.replace(/\D/g, '').slice(0, 16))}
            className="font-mono"
          />
        </div>
        <div className="grid grid-cols-2 gap-4">
          <Input
            placeholder="MM/YY"
            value={expiry}
            onChange={(e) => {
              let value = e.target.value.replace(/\D/g, '');
              if (value.length >= 2) {
                value = value.slice(0, 2) + '/' + value.slice(2, 4);
              }
              setExpiry(value);
            }}
            maxLength={5}
          />
          <Input
            placeholder="CVV"
            value={cvv}
            onChange={(e) => setCvv(e.target.value.replace(/\D/g, '').slice(0, 3))}
            maxLength={3}
            type="password"
          />
        </div>
        {error && <p className="text-red-500 text-sm">{error}</p>}
        <Button type="submit" className="w-full" disabled={processing}>
          {processing ? 'Processing...' : `Pay $${amount.toFixed(2)}`}
        </Button>
      </form>
    </div>
  );
};

const ManagementDashboard = () => {
  const [formData, setFormData] = useState({
    firstName: '',
    middleName: '',
    lastName: '',
    phoneCode: '1',
    phoneNumber: '',
    email: '',
    address: '',
    state: '',
    city: '',
    zipCode: ''
  });
  
  const [shares, setShares] = useState(1);
  const [totalAmount, setTotalAmount] = useState(SHARE_PRICE);
  const [cities, setCities] = useState([]);
  const [errors, setErrors] = useState({});
  const [registrationComplete, setRegistrationComplete] = useState(false);
  const [paymentComplete, setPaymentComplete] = useState(false);
  const [din, setDin] = useState('');
  const [certificates, setCertificates] = useState([]);
  const [activities, setActivities] = useState([]);
  const [lastDinSequence, setLastDinSequence] = useState(1);
  const [lastCertSequence, setLastCertSequence] = useState(1);
  const [showPayment, setShowPayment] = useState(false);

  useEffect(() => {
    setTotalAmount(shares * SHARE_PRICE);
  }, [shares]);

  useEffect(() => {
    if (formData.state) {
      setCities(USA_STATES[formData.state]?.cities || []);
    }
  }, [formData.state]);

  const validateForm = () => {
    const newErrors = {};
    if (!formData.firstName) newErrors.firstName = 'First name is required';
    if (!formData.lastName) newErrors.lastName = 'Last name is required';
    if (!formData.phoneNumber) newErrors.phoneNumber = 'Phone number is required';
    if (!/^\d{10}$/.test(formData.phoneNumber)) newErrors.phoneNumber = 'Invalid phone number';
    if (!formData.email) newErrors.email = 'Email is required';
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(formData.email)) newErrors.email = 'Invalid email';
    if (!formData.state) newErrors.state = 'State is required';
    if (!formData.city) newErrors.city = 'City is required';
    if (!formData.zipCode) newErrors.zipCode = 'ZIP code is required';
    if (formData.state && !new RegExp(USA_STATES[formData.state].zipCodePattern).test(formData.zipCode)) {
      newErrors.zipCode = `Invalid ZIP code for ${USA_STATES[formData.state].name}`;
    }
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    if (validateForm()) {
      setRegistrationComplete(true);
      setShowPayment(true);
      addActivity('Registration completed');
    }
  };

  const handlePaymentSuccess = () => {
    setPaymentComplete(true);
    setShowPayment(false);
    addActivity(`Payment processed: $${totalAmount.toFixed(2)} for ${shares} shares`);
  };

  const generateDIN = () => {
    const countryCode = formData.phoneCode.padStart(3, '0');
    const sequence = lastDinSequence.toString().padStart(7, '0');
    const newDin = `${countryCode}${sequence}`;
    setDin(newDin);
    setLastDinSequence(prev => prev + 1);
    addActivity('DIN generated');
  };

  const generateCertificates = () => {
    const newCertificates = Array(shares).fill(null).map((_, index) => {
      const sequence = (lastCertSequence + index).toString().padStart(3, '0');
      return `SEC${sequence}`;
    });
    setCertificates(newCertificates);
    setLastCertSequence(prev => prev + shares);
    addActivity('Stock certificates generated');
  };

  const addActivity = (description) => {
    const timestamp = new Date();
    setActivities(prev => [...prev, {
      description,
      timestamp,
      formattedTime: new Intl.DateTimeFormat('en-US', {
        year: 'numeric',
        month: 'short',
        day: '2-digit',
        hour: '2-digit',
        minute: '2-digit',
        second: '2-digit'
      }).format(timestamp)
    }]);
  };

  // Demo data loader
  const loadDemoData = () => {
    setFormData({
      firstName: 'John',
      middleName: 'Robert',
      lastName: 'Doe',
      phoneCode: '1',
      phoneNumber: '4155552671',
      email: 'john.doe@example.com',
      address: '123 Market Street',
      state: 'CA',
      city: 'San Francisco',
      zipCode: '94105'
    });
    setShares(3);
  };

  return (
    <div className="container mx-auto p-4">
      <Tabs defaultValue="registration" className="w-full">
        <TabsList className="grid w-full grid-cols-2">
          <TabsTrigger value="registration">Registration & Payment</TabsTrigger>
          <TabsTrigger value="management" disabled={!paymentComplete}>
            DIN & Stock Management
          </TabsTrigger>
        </TabsList>

        <TabsContent value="registration">
          <Card>
            <CardContent className="pt-6">
              {!showPayment ? (
                <form onSubmit={handleSubmit} className="space-y-6">
                  <div className="flex justify-end">
                    <Button type="button" variant="outline" onClick={loadDemoData}>
                      Load Demo Data
                    </Button>
                  </div>
                  
                  <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                    <div>
                      <Input
                        placeholder="First Name"
                        value={formData.firstName}
                        onChange={e => setFormData({...formData, firstName: e.target.value})}
                        className={errors.firstName ? "border-red-500" : ""}
                      />
                      {errors.firstName && <span className="text-red-500 text-sm">{errors.firstName}</span>}
                    </div>
                    <Input
                      placeholder="Middle Name (Optional)"
                      value={formData.middleName}
                      onChange={e => setFormData({...formData, middleName: e.target.value})}
                    />
                    <div>
                      <Input
                        placeholder="Last Name"
                        value={formData.lastName}
                        onChange={e => setFormData({...formData, lastName: e.target.value})}
                        className={errors.lastName ? "border-red-500" : ""}
                      />
                      {errors.lastName && <span className="text-red-500 text-sm">{errors.lastName}</span>}
                    </div>
                  </div>

                  <div className="space-y-4">
                    <div className="grid grid-cols-3 gap-4">
                      <Select
                        value={formData.phoneCode}
                        onValueChange={(value) => setFormData({...formData, phoneCode: value})}
                      >
                        <SelectTrigger>
                          <SelectValue placeholder="Code" />
                        </SelectTrigger>
                        <SelectContent>
                          <SelectItem value="1">+1</SelectItem>
                        </SelectContent>
                      </Select>
                      <div className="col-span-2">
                        <Input
                          placeholder="Phone Number"
                          value={formData.phoneNumber}
                          onChange={e => setFormData({...formData, phoneNumber: e.target.value.replace(/\D/g, '')})}
                          className={errors.phoneNumber ? "border-red-500" : ""}
                          maxLength={10}
                        />
                        {errors.phoneNumber && <span className="text-red-500 text-sm">{errors.phoneNumber}</span>}
                      </div>
                    </div>

                    <div>
                      <Input
                        placeholder="Email Address"
                        type="email"
                        value={formData.email}
                        onChange={e => setFormData({...formData, email: e.target.value})}
                        className={errors.email ? "border-red-500" : ""}
                      />
                      {errors.email && <span className="text-red-500 text-sm">{errors.email}</span>}
                    </div>

                    <Input
                      placeholder="Address"
                      value={formData.address}
                      onChange={e => setFormData({...formData, address: e.target.value})}
                    />

                    <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                      <Select
                        value={formData.state}
                        onValueChange={(value) => setFormData({...formData, state: value, city: ''})}
                      >
                        <SelectTrigger>
                          <SelectValue placeholder="Select State" />
                        </SelectTrigger>
                        <SelectContent>
                          {Object.entries(USA_STATES).map(([code, state]) => (
                            <SelectItem key={code} value={code}>{state.name}</SelectItem>
                          ))}
                        </SelectContent>
                      </Select>

                      <Select
                        value={formData.city}
                        onValueChange={(value) => setFormData({...formData, city: value})}
                        disabled={!formData.state}
                      >
                        <SelectTrigger>
                          <SelectValue placeholder="Select City" />
                        </SelectTrigger>
                        <SelectContent>
                          {cities.map(city => (
                            <SelectItem key={city} value={city}>{city}</SelectItem>
                          ))}
                        </SelectContent>
                      </Select>

                      <div>
                        <Input
                          placeholder="ZIP Code"
                          value={formData.zipCode}
                          onChange={e => setFormData({...formData, zipCode: e.target.value})}
                          className={errors.zipCode ? "border-red-500" : ""}
                        />
                        {errors.zipCode && <span className="text-red-500 text-sm">{errors.zipCode}</span>}
                      </div>
                    </div>
                  </div>

                  <div className="space-y-4 border-t pt-4">
                    <h3 className="text-lg font-medium">Share Purchase</h3>
                    <div className="grid grid-cols-2 gap-4">
                      <Select 
                        value={shares.toString()} 
                        onValueChange={(value) => setShares(parseInt(value))}
                      >
                        <SelectTrigger>
                          <SelectValue placeholder="Select shares" />
                        </SelectTrigger>
                        <SelectContent>
                          {Array.from({length: MAX_SHARES}, (_, i) => i + 1).map(num => (
                            <SelectItem key={num} value={num.toString()}>
                             
                        {num} Share{num > 1 ? 's' : ''} (${(num * SHARE_PRICE).toFixed(2)})
                            </SelectItem>
                          ))}
                        </SelectContent>
                      </Select>
                      <Input 
                        value={`$${totalAmount.toFixed(2)}`} 
                        disabled 
                        className="bg-gray-50 font-medium"
                      />
                    </div>
                    <Button type="submit" className="w-full">Proceed to Payment</Button>
                  </div>
                </form>
              ) : (
                <div className="space-y-6">
                  <div className="bg-gray-50 p-4 rounded-lg">
                    <h3 className="font-medium mb-2">Order Summary</h3>
                    <div className="space-y-1 text-sm">
                      <p>Shares: {shares}</p>
                      <p>Price per Share: ${SHARE_PRICE.toFixed(2)}</p>
                      <p className="font-medium">Total Amount: ${totalAmount.toFixed(2)}</p>
                    </div>
                  </div>
                  <PaymentSection 
                    amount={totalAmount} 
                    onSuccess={handlePaymentSuccess}
                  />
                </div>
              )}
            </CardContent>
          </Card>
        </TabsContent>

        <TabsContent value="management">
          <Card>
            <CardContent className="space-y-6 pt-6">
              <div className="bg-gray-50 p-4 rounded-lg">
                <h3 className="font-medium mb-2">User Details</h3>
                <div className="space-y-1 text-sm">
                  <p>Name: {formData.firstName} {formData.middleName} {formData.lastName}</p>
                  <p>Email: {formData.email}</p>
                  <p>Phone: +{formData.phoneCode} {formData.phoneNumber}</p>
                  <p>Address: {formData.address}, {formData.city}, {USA_STATES[formData.state]?.name}, {formData.zipCode}</p>
                  <p>Shares Purchased: {shares}</p>
                  <p>Total Investment: ${totalAmount.toFixed(2)}</p>
                </div>
              </div>

              <div className="border-t pt-4">
                <h3 className="font-medium mb-4">DIN Management</h3>
                {din ? (
                  <Alert>
                    <AlertDescription className="font-mono">{din}</AlertDescription>
                  </Alert>
                ) : (
                  <Button onClick={generateDIN}>Generate DIN</Button>
                )}
              </div>

              <div className="border-t pt-4">
                <h3 className="font-medium mb-4">Stock Certificates</h3>
                {certificates.length > 0 ? (
                  <div className="space-y-2">
                    {certificates.map((cert, index) => (
                      <Alert key={index}>
                        <AlertDescription className="font-mono">{cert}</AlertDescription>
                      </Alert>
                    ))}
                  </div>
                ) : (
                  <Button onClick={generateCertificates}>Generate Certificates</Button>
                )}
              </div>

              <div className="border-t pt-4">
                <h3 className="font-medium mb-4">Activity Timeline</h3>
                <div className="space-y-2">
                  {activities.map((activity, index) => (
                    <div key={index} className="flex items-center justify-between p-2 bg-gray-50 rounded">
                      <span className="text-sm">{activity.description}</span>
                      <span className="text-xs text-gray-500">{activity.formattedTime}</span>
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

I've enhanced the dashboard with the following features:

1. Share Price Calculation:
   - Fixed price of $100 per share
   - Automatic total calculation
   - Share selection shows price in dropdown
   - Real-time total amount updates

2. Payment Processing:
   - Dedicated payment section after registration
   - Credit card validation
   - CVV and expiry date formatting
   - Payment processing simulation
   - Clear error messages

3. Demo Functionality:
   - "Load Demo Data" button
   - Pre-fills form with sample data
   - Makes testing easier

4. Enhanced User Flow:
   - Registration → Payment → Management
   - Clear progression between stages
   - Proper validation at each step
   - Informative order summary

5. Additional Features:
   - Phone number formatting
   - ZIP code validation per state
   - Real-time error checking
   - Comprehensive activity logging

To test the dashboard:
1. Click "Load Demo Data" or fill out the form
2. Select number of shares (price updates automatically)
3. Submit registration form
4. Enter payment details (any 16-digit number, MM/YY format, 3-digit CVV)
5. After payment, access DIN & Stock Management
6. Generate DIN and certificates

Would you like me to modify any part of the implementation or add additional features?
