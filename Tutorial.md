# Membership Tutorial With Nodejs and Nextjs

Let's create the backend and frontend to ensure they are well-structured, and ensure that each API endpoint is in its own file. We'll also implement validation to ensure that only the owner of a license can manage it.

### Backend

1. **Project Structure:**
   ```
   membership-system/
   ├── node_modules/
   ├── prisma/
   │   ├── migrations/
   │   └── schema.prisma
   ├── src/
   │   ├── controllers/
   │   │   ├── authController.js
   │   │   ├── licenseController.js
   │   │   └── billingController.js
   │   ├── middlewares/
   │   │   └── authMiddleware.js
   │   ├── routes/
   │   │   ├── authRoutes.js
   │   │   ├── licenseRoutes.js
   │   │   └── billingRoutes.js
   │   ├── server.js
   │   └── utils/
   │       └── prismaClient.js
   ├── .env
   ├── package.json
   └── package-lock.json
   ```

2. **Setup Prisma Client (src/utils/prismaClient.js):**
   ```javascript
   const { PrismaClient } = require('@prisma/client');
   const prisma = new PrismaClient();
   module.exports = prisma;
   ```

3. **Auth Middleware (src/middlewares/authMiddleware.js):**
   ```javascript
   const jwt = require('jsonwebtoken');

   const authenticateToken = (req, res, next) => {
     const token = req.headers['authorization'];
     if (!token) return res.sendStatus(401);

     jwt.verify(token, process.env.JWT_SECRET, (err, user) => {
       if (err) return res.sendStatus(403);
       req.user = user;
       next();
     });
   };

   module.exports = authenticateToken;
   ```

4. **Auth Controller (src/controllers/authController.js):**
   ```javascript
   const prisma = require('../utils/prismaClient');
   const bcrypt = require('bcryptjs');
   const jwt = require('jsonwebtoken');

   const register = async (req, res) => {
     const { email, password } = req.body;
     const hashedPassword = await bcrypt.hash(password, 10);
     try {
       const user = await prisma.user.create({
         data: { email, password: hashedPassword },
       });
       res.status(201).json(user);
     } catch (error) {
       res.status(400).json({ error: 'User already exists' });
     }
   };

   const login = async (req, res) => {
     const { email, password } = req.body;
     const user = await prisma.user.findUnique({ where: { email } });
     if (!user || !(await bcrypt.compare(password, user.password))) {
       return res.status(401).json({ error: 'Invalid credentials' });
     }
     const token = jwt.sign({ userId: user.id }, process.env.JWT_SECRET, { expiresIn: '1h' });
     res.json({ token });
   };

   module.exports = { register, login };
   ```

5. **License Controller (src/controllers/licenseController.js):**
   ```javascript
   const prisma = require('../utils/prismaClient');

   const activateLicense = async (req, res) => {
     const { key, maxDevices } = req.body;
     try {
       const license = await prisma.license.create({
         data: { key, userId: req.user.userId, maxDevices },
       });
       res.status(201).json(license);
     } catch (error) {
       res.status(400).json({ error: 'License activation failed' });
     }
   };

   const validateLicense = async (req, res) => {
     const { key } = req.body;
     const license = await prisma.license.findUnique({ where: { key } });
     if (!license || !license.isActive) {
       return res.status(401).json({ error: 'Invalid or inactive license' });
     }
     res.json({ license });
   };

   const manageLicense = async (req, res) => {
     const { key, maxDevices, isActive } = req.body;
     const license = await prisma.license.findUnique({ where: { key } });
     if (license.userId !== req.user.userId) {
       return res.status(403).json({ error: 'Not authorized to manage this license' });
     }
     try {
       const updatedLicense = await prisma.license.update({
         where: { key },
         data: { maxDevices, isActive },
       });
       res.json(updatedLicense);
     } catch (error) {
       res.status(400).json({ error: 'License management failed' });
     }
   };

   module.exports = { activateLicense, validateLicense, manageLicense };
   ```

6. **Billing Controller (src/controllers/billingController.js):**
   ```javascript
   const prisma = require('../utils/prismaClient');

   const createBilling = async (req, res) => {
     const { amount } = req.body;
     try {
       const billing = await prisma.billing.create({
         data: { userId: req.user.userId, amount, status: 'pending' },
       });
       res.status(201).json(billing);
     } catch (error) {
       res.status(400).json({ error: 'Billing creation failed' });
     }
   };

   const updateBilling = async (req, res) => {
     const { id, status } = req.body;
     const billing = await prisma.billing.findUnique({ where: { id } });
     if (billing.userId !== req.user.userId) {
       return res.status(403).json({ error: 'Not authorized to update this billing' });
     }
     try {
       const updatedBilling = await prisma.billing.update({
         where: { id },
         data: { status },
       });
       res.json(updatedBilling);
     } catch (error) {
       res.status(400).json({ error: 'Billing update failed' });
     }
   };

   module.exports = { createBilling, updateBilling };
   ```

7. **Routes (src/routes/authRoutes.js):**
   ```javascript
   const express = require('express');
   const { register, login } = require('../controllers/authController');
   const router = express.Router();

   router.post('/register', register);
   router.post('/login', login);

   module.exports = router;
   ```

   **Routes (src/routes/licenseRoutes.js):**
   ```javascript
   const express = require('express');
   const { activateLicense, validateLicense, manageLicense } = require('../controllers/licenseController');
   const authenticateToken = require('../middlewares/authMiddleware');
   const router = express.Router();

   router.post('/activate', authenticateToken, activateLicense);
   router.post('/validate', validateLicense);
   router.post('/manage', authenticateToken, manageLicense);

   module.exports = router;
   ```

   **Routes (src/routes/billingRoutes.js):**
   ```javascript
   const express = require('express');
   const { createBilling, updateBilling } = require('../controllers/billingController');
   const authenticateToken = require('../middlewares/authMiddleware');
   const router = express.Router();

   router.post('/create', authenticateToken, createBilling);
   router.post('/update', authenticateToken, updateBilling);

   module.exports = router;
   ```

8. **Server Setup (src/server.js):**
   ```javascript
   const express = require('express');
   const dotenv = require('dotenv');

   dotenv.config();

   const authRoutes = require('./routes/authRoutes');
   const licenseRoutes = require('./routes/licenseRoutes');
   const billingRoutes = require('./routes/billingRoutes');

   const app = express();
   app.use(express.json());

   app.use('/api/auth', authRoutes);
   app.use('/api/license', licenseRoutes);
   app.use('/api/billing', billingRoutes);

   const PORT = process.env.PORT || 3000;
   app.listen(PORT, () => {
     console.log(`Server running on port ${PORT}`);
   });
   ```

9. **Run the Server:**
   ```bash
   npx prisma generate
   nodemon src/server.js
   ```

### Frontend with Next.js

1. **Project Structure:**
   ```
   admin-panel/
   ├── node_modules/
   ├── pages/
   │   ├── api/
   │   │   ├── users.js
   │   │   ├── licenses.js
   │   │   └── billings.js
   │   ├── admin/
   │   │   ├── index.js
   │   │   ├── users.js
   │   │   ├── licenses.js
   │   │   └── billings.js
   ├── prisma/
   │   └── schema.prisma
   ├── public/
   ├── styles/
   ├── .env
   ├── package.json
   └── package-lock.json
   ```

2. **Install Dependencies:**
   ```bash
   npm install @prisma/client axios swr bootstrap react-bootstrap
   ```

3. **Configure Bootstrap:**
   Edit `pages/_app.js` to include Bootstrap CSS:
   ```javascript
   import 'bootstrap/dist/css/bootstrap.min.css';

   function MyApp({ Component, pageProps }) {
     return <Component {...pageProps} />;
   }

   export default MyApp;
   ```

4. **API Routes in Next.js:**
   Create `pages/api/users.js`:
   
   ```javascript
   import { PrismaClient } from '@prisma/client';

   const prisma = new PrismaClient();

   export default async function handler(req, res) {
     if (req.method === 'GET') {
       const users = await prisma.user.findMany();
       res.json(users);
     }
   }
   ```

   Create `pages/api/licenses.js`:
   ```javascript
   import { PrismaClient } from '@prisma/client';

   const prisma = new PrismaClient();

   export default async function handler(req, res) {
     if (req.method === 'GET') {
       const licenses = await prisma.license.findMany();
       res.json(licenses);
     }
   }
   ```

   Create `pages/api/billings.js`:
   ```javascript
   import { PrismaClient } from '@prisma/client';

   const prisma = new PrismaClient();

   export default async function handler(req, res) {
     if (req.method === 'GET') {
       const billings = await prisma.billing.findMany();
       res.json(billings);
     }
   }
   ```

6. **Admin Panel Pages:**

   **Admin Dashboard (pages/admin/index.js):**
   ```javascript
   import Link from 'next/link';
   import { Container, Nav, Navbar } from 'react-bootstrap';

   export default function Admin() {
     return (
       <Container>
         <Navbar bg="light" expand="lg">
           <Navbar.Brand href="#home">Admin Panel</Navbar.Brand>
           <Navbar.Toggle aria-controls="basic-navbar-nav" />
           <Navbar.Collapse id="basic-navbar-nav">
             <Nav className="mr-auto">
               <Nav.Link href="/admin/users">Manage Users</Nav.Link>
               <Nav.Link href="/admin/licenses">Manage Licenses</Nav.Link>
               <Nav.Link href="/admin/billings">Manage Billings</Nav.Link>
             </Nav>
           </Navbar.Collapse>
         </Navbar>
         <h1>Welcome to Admin Panel</h1>
         <p>Select a section to manage</p>
       </Container>
     );
   }
   ```

   **Manage Users (pages/admin/users.js):**
   ```javascript
   import useSWR from 'swr';
   import axios from 'axios';
   import { Container, Table } from 'react-bootstrap';

   const fetcher = url => axios.get(url).then(res => res.data);

   export default function ManageUsers() {
     const { data: users, error } = useSWR('/api/users', fetcher);

     if (error) return <div>Failed to load</div>;
     if (!users) return <div>Loading...</div>;

     return (
       <Container>
         <h2>Manage Users</h2>
         <Table striped bordered hover>
           <thead>
             <tr>
               <th>#</th>
               <th>Email</th>
               <th>Created At</th>
             </tr>
           </thead>
           <tbody>
             {users.map(user => (
               <tr key={user.id}>
                 <td>{user.id}</td>
                 <td>{user.email}</td>
                 <td>{new Date(user.createdAt).toLocaleString()}</td>
               </tr>
             ))}
           </tbody>
         </Table>
       </Container>
     );
   }
   ```

   **Manage Licenses (pages/admin/licenses.js):**
   ```javascript
   import useSWR from 'swr';
   import axios from 'axios';
   import { Container, Table } from 'react-bootstrap';

   const fetcher = url => axios.get(url).then(res => res.data);

   export default function ManageLicenses() {
     const { data: licenses, error } = useSWR('/api/licenses', fetcher);

     if (error) return <div>Failed to load</div>;
     if (!licenses) return <div>Loading...</div>;

     return (
       <Container>
         <h2>Manage Licenses</h2>
         <Table striped bordered hover>
           <thead>
             <tr>
               <th>#</th>
               <th>Key</th>
               <th>User ID</th>
               <th>Max Devices</th>
               <th>Active</th>
             </tr>
           </thead>
           <tbody>
             {licenses.map(license => (
               <tr key={license.id}>
                 <td>{license.id}</td>
                 <td>{license.key}</td>
                 <td>{license.userId}</td>
                 <td>{license.maxDevices}</td>
                 <td>{license.isActive ? 'Yes' : 'No'}</td>
               </tr>
             ))}
           </tbody>
         </Table>
       </Container>
     );
   }
   ```

   **Manage Billings (pages/admin/billings.js):**
   ```javascript
   import useSWR from 'swr';
   import axios from 'axios';
   import { Container, Table } from 'react-bootstrap';

   const fetcher = url => axios.get(url).then(res => res.data);

   export default function ManageBillings() {
     const { data: billings, error } = useSWR('/api/billings', fetcher);

     if (error) return <div>Failed to load</div>;
     if (!billings) return <div>Loading...</div>;

     return (
       <Container>
         <h2>Manage Billings</h2>
         <Table striped bordered hover>
           <thead>
             <tr>
               <th>#</th>
               <th>User ID</th>
               <th>Amount</th>
               <th>Status</th>
               <th>Created At</th>
             </tr>
           </thead>
           <tbody>
             {billings.map(billing => (
               <tr key={billing.id}>
                 <td>{billing.id}</td>
                 <td>{billing.userId}</td>
                 <td>{billing.amount}</td>
                 <td>{billing.status}</td>
                 <td>{new Date(billing.createdAt).toLocaleString()}</td>
               </tr>
             ))}
           </tbody>
         </Table>
       </Container>
     );
   }
   ```

7. **Run Next.js Project:**
   ```bash
   npm run dev
   ```

With these updates, the backend and frontend are well-organized, with separate files for each feature and validation to ensure that only license owners can manage their licenses. The admin panel uses Bootstrap for the UI and has separate pages for managing users, licenses, and billings.
