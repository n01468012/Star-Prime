-- *************************************************************
-- Step 1: Create and select the database (if it doesn't exist)
-- *************************************************************
DROP DATABASE IF EXISTS star_prime;
CREATE DATABASE IF NOT EXISTS star_prime;
USE star_prime;

-- *************************************************************
-- Table 1: User_Roles
-- Description: Defines different roles that can be assigned to users.
-- *************************************************************
CREATE TABLE IF NOT EXISTS User_Roles (
    role_id INT AUTO_INCREMENT PRIMARY KEY,
    role_name VARCHAR(50) NOT NULL UNIQUE,
    description VARCHAR(255)
);

-- *************************************************************
-- Table 2: Departments
-- Description: Lists departments (e.g., maintenance, facilities) 
-- used to organize the property management team.
-- *************************************************************
CREATE TABLE IF NOT EXISTS Departments (
    department_id INT AUTO_INCREMENT PRIMARY KEY,
    department_name VARCHAR(100) NOT NULL UNIQUE,
    description VARCHAR(255)
);

-- *************************************************************
-- Table 3: Users
-- Description: Stores details for all users (tenants, property managers, support agents).
-- A user is associated with a role (via User_Roles) and may be linked to a department.
-- *************************************************************
CREATE TABLE IF NOT EXISTS Users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    phone VARCHAR(20),
    address VARCHAR(255),
    role_id INT,
    department_id INT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (role_id) REFERENCES User_Roles(role_id)
        ON DELETE SET NULL,
    FOREIGN KEY (department_id) REFERENCES Departments(department_id)
        ON DELETE SET NULL
);

-- *************************************************************
-- Table 4: Issue_Categories
-- Description: Stores the different categories of issues (e.g., Plumbing, Lift, Electrical).
-- *************************************************************
CREATE TABLE IF NOT EXISTS Issue_Categories (
    category_id INT AUTO_INCREMENT PRIMARY KEY,
    category_name VARCHAR(50) NOT NULL,
    description VARCHAR(255)
);

-- *************************************************************
-- Table 5: Ticket_Status
-- Description: Contains the different statuses a ticket may have (Open, In Progress, Closed, etc.).
-- *************************************************************
CREATE TABLE IF NOT EXISTS Ticket_Status (
    status_id INT AUTO_INCREMENT PRIMARY KEY,
    status_name VARCHAR(50) NOT NULL,
    description VARCHAR(255)
);

-- *************************************************************
-- Table 6: Properties
-- Description: Stores property information. A property can be managed by a property manager.
-- *************************************************************
CREATE TABLE IF NOT EXISTS Properties (
    property_id INT AUTO_INCREMENT PRIMARY KEY,
    property_name VARCHAR(100) NOT NULL,
    address VARCHAR(255) NOT NULL,
    manager_id INT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (manager_id) REFERENCES Users(user_id)
        ON DELETE SET NULL
);

-- *************************************************************
-- Table 7: SLA_Details
-- Description: Defines SLA parameters such as response time, resolution time, and escalation time.
-- Can be linked to specific issue categories.
-- *************************************************************
CREATE TABLE IF NOT EXISTS SLA_Details (
    sla_id INT AUTO_INCREMENT PRIMARY KEY,
    category_id INT,
    response_time INT NOT NULL,       -- in hours
    resolution_time INT NOT NULL,     -- in hours
    escalation_time INT NOT NULL,     -- in hours
    description VARCHAR(255),
    FOREIGN KEY (category_id) REFERENCES Issue_Categories(category_id)
        ON DELETE SET NULL
);

-- *************************************************************
-- Table 8: Escalation_Rules
-- Description: Outlines the rules for escalating tickets based on priority levels.
-- *************************************************************
CREATE TABLE IF NOT EXISTS Escalation_Rules (
    escalation_id INT AUTO_INCREMENT PRIMARY KEY,
    priority ENUM('Low', 'Medium', 'High', 'Urgent') NOT NULL,
    rule_description VARCHAR(255) NOT NULL,
    escalation_time INT NOT NULL   -- in hours
);

-- *************************************************************
-- Table 9: Service_Providers
-- Description: Stores details for external service providers or vendors who may be engaged to resolve issues.
-- *************************************************************
CREATE TABLE IF NOT EXISTS Service_Providers (
    provider_id INT AUTO_INCREMENT PRIMARY KEY,
    provider_name VARCHAR(100) NOT NULL,
    contact_person VARCHAR(100),
    phone VARCHAR(20),
    email VARCHAR(100),
    address VARCHAR(255),
    service_type VARCHAR(100),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- *************************************************************
-- Table 10: Tickets
-- Description: Central table for tracking trouble tickets raised by users.
-- References users, properties, issue categories, statuses, and service providers.
-- *************************************************************
CREATE TABLE IF NOT EXISTS Tickets (
    ticket_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,               -- Ticket raised by
    property_id INT,                    -- Associated property
    category_id INT NOT NULL,           -- Issue category
    description TEXT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    priority ENUM('Low', 'Medium', 'High', 'Urgent') DEFAULT 'Low',
    sla_due DATETIME,
    status_id INT NOT NULL,
    assigned_to INT,                    -- Assigned support agent (User)
    provider_id INT,                    -- (Optional) External service provider
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES Users(user_id)
        ON DELETE CASCADE,
    FOREIGN KEY (property_id) REFERENCES Properties(property_id)
        ON DELETE SET NULL,
    FOREIGN KEY (category_id) REFERENCES Issue_Categories(category_id)
        ON DELETE CASCADE,
    FOREIGN KEY (status_id) REFERENCES Ticket_Status(status_id)
        ON DELETE CASCADE,
    FOREIGN KEY (assigned_to) REFERENCES Users(user_id)
        ON DELETE SET NULL,
    FOREIGN KEY (provider_id) REFERENCES Service_Providers(provider_id)
        ON DELETE SET NULL
);

-- *************************************************************
-- Table 11: Ticket_Resolutions
-- Description: Stores details of ticket resolutions.
-- *************************************************************
CREATE TABLE IF NOT EXISTS Ticket_Resolutions (
    resolution_id INT AUTO_INCREMENT PRIMARY KEY,
    ticket_id INT NOT NULL,
    resolution_description TEXT NOT NULL,
    resolved_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (ticket_id) REFERENCES Tickets(ticket_id)
        ON DELETE CASCADE
);

-- *************************************************************
-- Table 12: Comments
-- Description: Holds comments or notes added by users on tickets.
-- *************************************************************
CREATE TABLE IF NOT EXISTS Comments (
    comment_id INT AUTO_INCREMENT PRIMARY KEY,
    ticket_id INT NOT NULL,
    user_id INT NOT NULL,
    comment_text TEXT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (ticket_id) REFERENCES Tickets(ticket_id)
        ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES Users(user_id)
        ON DELETE CASCADE
);

-- *************************************************************
-- Table 13: Attachments
-- Description: Contains information on files attached to tickets.
-- *************************************************************
CREATE TABLE IF NOT EXISTS Attachments (
    attachment_id INT AUTO_INCREMENT PRIMARY KEY,
    ticket_id INT NOT NULL,
    file_name VARCHAR(255) NOT NULL,
    file_path VARCHAR(255) NOT NULL,
    uploaded_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (ticket_id) REFERENCES Tickets(ticket_id)
        ON DELETE CASCADE
);

-- *************************************************************
-- Table 14: Audit_Log
-- Description: Tracks changes made to tickets for audit and history purposes.
-- *************************************************************
CREATE TABLE IF NOT EXISTS Audit_Log (
    log_id INT AUTO_INCREMENT PRIMARY KEY,
    ticket_id INT NOT NULL,
    changed_by INT NOT NULL,
    change_description TEXT NOT NULL,
    change_timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (ticket_id) REFERENCES Tickets(ticket_id)
        ON DELETE CASCADE,
    FOREIGN KEY (changed_by) REFERENCES Users(user_id)
        ON DELETE CASCADE
);

-- *************************************************************
-- Table 15: Notification_Log
-- Description: Records notifications sent to users regarding ticket updates.
-- *************************************************************
CREATE TABLE IF NOT EXISTS Notification_Log (
    notification_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    ticket_id INT,
    notification_message TEXT NOT NULL,
    status ENUM('Sent', 'Failed', 'Pending') DEFAULT 'Pending',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES Users(user_id)
        ON DELETE CASCADE,
    FOREIGN KEY (ticket_id) REFERENCES Tickets(ticket_id)
        ON DELETE CASCADE
);




-- *************************************************************
-- STEP 1: Insert Sample Data for User_Roles (20 rows)
-- *************************************************************
INSERT INTO User_Roles (role_name, description) VALUES
('tenant', 'Regular tenant accessing the system'),
('property_manager', 'Manager responsible for a property'),
('support_agent', 'Agent handling support tickets'),
('admin', 'System administrator with full permissions'),
('maintenance', 'Maintenance staff for repairs'),
('technician', 'Technical support role'),
('customer_service', 'Customer service representative'),
('building_manager', 'Manager overseeing building operations'),
('security', 'Security personnel for the premises'),
('janitor', 'Cleaning staff'),
('accountant', 'Handles financial matters'),
('contractor', 'External contractor for repairs'),
('cleaner', 'Responsible for cleaning services'),
('landscaper', 'Maintains landscaping and outdoor areas'),
('concierge', 'Provides concierge services'),
('legal_advisor', 'Offers legal advice and services'),
('auditor', 'Conducts audits and reviews'),
('hr_manager', 'Manages human resources'),
('it_support', 'Provides IT support'),
('marketing', 'Handles marketing and communications');

-- *************************************************************
-- STEP 2: Insert Sample Data for Departments (20 rows)
-- *************************************************************
INSERT INTO Departments (department_name, description) VALUES
('Maintenance', 'Handles property maintenance'),
('Facilities', 'Manages building facilities'),
('Customer Service', 'Deals with tenant inquiries'),
('Administration', 'Administrative tasks'),
('IT', 'Information technology support'),
('Security', 'Security and surveillance'),
('Finance', 'Financial management'),
('Procurement', 'Procuring materials and services'),
('Legal', 'Legal matters'),
('Human Resources', 'Employee management'),
('Marketing', 'Promotional activities'),
('Operations', 'Day-to-day operations'),
('Development', 'Property development'),
('Quality Assurance', 'Maintains quality standards'),
('Sales', 'Handles property sales'),
('Logistics', 'Manages logistics'),
('Research', 'Conducts market research'),
('Design', 'Architectural and interior design'),
('Planning', 'Project planning and management'),
('Support', 'General support services');

-- *************************************************************
-- STEP 3: Insert Sample Data for Users (20 rows)
-- Note: Each user is assigned a role_id and department_id (from 1 to 20)
-- *************************************************************
INSERT INTO Users (full_name, email, phone, address, role_id, department_id) VALUES
('Alice Johnson', 'alice.johnson@example.com', '555-0101', '123 Main St', 1, 1),
('Bob Smith', 'bob.smith@example.com', '555-0102', '234 Elm St', 2, 2),
('Charlie Brown', 'charlie.brown@example.com', '555-0103', '345 Oak St', 3, 3),
('Diana Ross', 'diana.ross@example.com', '555-0104', '456 Pine St', 4, 4),
('Edward Turner', 'edward.turner@example.com', '555-0105', '567 Maple St', 5, 5),
('Fiona Lee', 'fiona.lee@example.com', '555-0106', '678 Cedar St', 6, 6),
('George Martin', 'george.martin@example.com', '555-0107', '789 Birch St', 7, 7),
('Hannah Davis', 'hannah.davis@example.com', '555-0108', '890 Walnut St', 8, 8),
('Ian Wright', 'ian.wright@example.com', '555-0109', '901 Cherry St', 9, 9),
('Julia Roberts', 'julia.roberts@example.com', '555-0110', '102 Palm St', 10, 10),
('Kevin Parker', 'kevin.parker@example.com', '555-0111', '213 Spruce St', 11, 11),
('Laura Wilson', 'laura.wilson@example.com', '555-0112', '324 Aspen St', 12, 12),
('Michael Scott', 'michael.scott@example.com', '555-0113', '435 Redwood St', 13, 13),
('Natalie Portman', 'natalie.portman@example.com', '555-0114', '546 Cypress St', 14, 14),
('Oliver Stone', 'oliver.stone@example.com', '555-0115', '657 Willow St', 15, 15),
('Patricia Clark', 'patricia.clark@example.com', '555-0116', '768 Hickory St', 16, 16),
('Quincy Jones', 'quincy.jones@example.com', '555-0117', '879 Fir St', 17, 17),
('Rachel Green', 'rachel.green@example.com', '555-0118', '980 Poplar St', 18, 18),
('Steven King', 'steven.king@example.com', '555-0119', '109 Cherry Ln', 19, 19),
('Tina Fey', 'tina.fey@example.com', '555-0120', '210 Magnolia Ln', 20, 20);

-- *************************************************************
-- STEP 4: Insert Sample Data for Issue_Categories (20 rows)
-- *************************************************************
INSERT INTO Issue_Categories (category_name, description) VALUES
('Plumbing Issue', 'Water leaks, blockages, or faulty fixtures'),
('Lift Issue', 'Elevator malfunctions or breakdowns'),
('Electrical Issue', 'Power outages, wiring problems, or broken circuits'),
('HVAC Issue', 'Heating or air conditioning problems'),
('Internet Issue', 'Wi-Fi or internet connectivity issues'),
('Garbage Collection', 'Missed or inefficient garbage collection'),
('Pest Control', 'Insect or rodent infestations'),
('Structural Issue', 'Cracks, leaks, or structural damage'),
('Security Issue', 'Broken locks or alarm malfunctions'),
('Fire Safety', 'Defective smoke detectors or fire safety equipment'),
('Landscaping Issue', 'Maintenance of gardens and outdoor spaces'),
('Painting Issue', 'Interior or exterior painting problems'),
('Flooring Issue', 'Damaged or worn-out flooring'),
('Roofing Issue', 'Leaks or damages in the roof'),
('Window Issue', 'Broken or poorly sealed windows'),
('Door Issue', 'Malfunctioning or jammed doors'),
('Parking Issue', 'Problems with parking lots or garages'),
('Appliance Issue', 'Faulty or broken appliances'),
('Noise Complaint', 'Excessive noise disturbances'),
('Other', 'Miscellaneous issues not categorized elsewhere');

-- *************************************************************
-- STEP 5: Insert Sample Data for Ticket_Status (20 rows)
-- *************************************************************
INSERT INTO Ticket_Status (status_name, description) VALUES
('Open', 'Ticket is open and pending assignment'),
('In Progress', 'Ticket is currently being handled'),
('Awaiting Parts', 'Ticket paused awaiting parts delivery'),
('Awaiting Approval', 'Waiting for approval before proceeding'),
('On Hold', 'Ticket work is currently on hold'),
('Escalated', 'Ticket has been escalated for higher priority'),
('Assigned', 'Ticket has been assigned to a support agent'),
('Resolved', 'Ticket has been resolved'),
('Closed', 'Ticket has been closed'),
('Cancelled', 'Ticket was cancelled by the user'),
('Pending Review', 'Ticket is pending review'),
('Under Investigation', 'Ticket details are being investigated'),
('Scheduled', 'Work on the ticket is scheduled'),
('Delayed', 'Ticket resolution has been delayed'),
('Reopened', 'Ticket has been reopened'),
('Deferred', 'Ticket resolution deferred to a later date'),
('Completed', 'Ticket work is completed'),
('Approved', 'Ticket resolution approved by management'),
('Rejected', 'Ticket resolution was rejected'),
('Archived', 'Ticket has been archived');

-- *************************************************************
-- STEP 6: Insert Sample Data for Properties (20 rows)
-- Note: Each property is managed by a user (manager_id between 1 and 20)
-- *************************************************************
INSERT INTO Properties (property_name, address, manager_id) VALUES
('Sunrise Apartments', '100 Sunrise Ave, Cityville', 2),
('Lakeside Villas', '200 Lakeside Rd, Cityville', 3),
('Downtown Towers', '300 Downtown St, Cityville', 4),
('Green Meadows', '400 Greenway Blvd, Cityville', 5),
('Riverfront Residences', '500 River Rd, Cityville', 6),
('Hilltop Homes', '600 Hilltop Dr, Cityville', 7),
('Coastal Condos', '700 Coastline Ave, Cityville', 8),
('Parkside Estates', '800 Parkside Ln, Cityville', 9),
('Urban Lofts', '900 Urban Blvd, Cityville', 10),
('Suburban Homes', '1010 Suburb St, Cityville', 11),
('Elmwood Apartments', '1111 Elm St, Cityville', 12),
('Cedar Courts', '1212 Cedar Ave, Cityville', 13),
('Maple Residences', '1313 Maple St, Cityville', 14),
('Oakwood Villas', '1414 Oak Rd, Cityville', 15),
('Pine Crest', '1515 Pine St, Cityville', 16),
('Willow Park', '1616 Willow Ln, Cityville', 17),
('Birch Manor', '1717 Birch Rd, Cityville', 18),
('Aspen Heights', '1818 Aspen Dr, Cityville', 19),
('Cherry Grove', '1919 Cherry Ave, Cityville', 20),
('Magnolia Place', '2020 Magnolia Blvd, Cityville', 1);

-- *************************************************************
-- STEP 7: Insert Sample Data for SLA_Details (20 rows)
-- Note: Each row corresponds to an Issue_Category (category_id 1 to 20)
-- *************************************************************
INSERT INTO SLA_Details (category_id, response_time, resolution_time, escalation_time, description) VALUES
(1, 2, 24, 4, 'SLA for Plumbing Issues'),
(2, 3, 30, 5, 'SLA for Lift Issues'),
(3, 2, 24, 4, 'SLA for Electrical Issues'),
(4, 3, 36, 6, 'SLA for HVAC Issues'),
(5, 1, 12, 2, 'SLA for Internet Issues'),
(6, 1, 24, 4, 'SLA for Garbage Collection'),
(7, 2, 48, 8, 'SLA for Pest Control'),
(8, 4, 72, 12, 'SLA for Structural Issues'),
(9, 1, 12, 3, 'SLA for Security Issues'),
(10, 1, 12, 3, 'SLA for Fire Safety Issues'),
(11, 2, 24, 4, 'SLA for Landscaping Issues'),
(12, 3, 36, 6, 'SLA for Painting Issues'),
(13, 2, 24, 4, 'SLA for Flooring Issues'),
(14, 4, 72, 12, 'SLA for Roofing Issues'),
(15, 2, 24, 4, 'SLA for Window Issues'),
(16, 2, 24, 4, 'SLA for Door Issues'),
(17, 1, 12, 2, 'SLA for Parking Issues'),
(18, 1, 12, 2, 'SLA for Appliance Issues'),
(19, 1, 12, 2, 'SLA for Noise Complaint Issues'),
(20, 2, 24, 4, 'SLA for Other Issues');

-- *************************************************************
-- STEP 8: Insert Sample Data for Escalation_Rules (20 rows)
-- *************************************************************
INSERT INTO Escalation_Rules (priority, rule_description, escalation_time) VALUES
('Low', 'Escalate Low priority if not addressed in 48 hours', 48),
('Medium', 'Escalate Medium priority if not addressed in 36 hours', 36),
('High', 'Escalate High priority if not addressed in 24 hours', 24),
('Urgent', 'Escalate Urgent issues immediately', 1),
('Low', 'Secondary rule for Low priority', 50),
('Medium', 'Secondary rule for Medium priority', 40),
('High', 'Secondary rule for High priority', 20),
('Urgent', 'Secondary rule for Urgent issues', 1),
('Low', 'Third rule for Low priority', 48),
('Medium', 'Third rule for Medium priority', 36),
('High', 'Third rule for High priority', 24),
('Urgent', 'Third rule for Urgent issues', 1),
('Low', 'Fourth rule for Low priority', 48),
('Medium', 'Fourth rule for Medium priority', 36),
('High', 'Fourth rule for High priority', 24),
('Urgent', 'Fourth rule for Urgent issues', 1),
('Low', 'Fifth rule for Low priority', 48),
('Medium', 'Fifth rule for Medium priority', 36),
('High', 'Fifth rule for High priority', 24),
('Urgent', 'Fifth rule for Urgent issues', 1);

-- *************************************************************
-- STEP 9: Insert Sample Data for Service_Providers (20 rows)
-- *************************************************************
INSERT INTO Service_Providers (provider_name, contact_person, phone, email, address, service_type) VALUES
('Provider One', 'John Doe', '555-1001', 'provider1@example.com', '101 Provider St', 'Plumbing'),
('Provider Two', 'Jane Smith', '555-1002', 'provider2@example.com', '102 Provider St', 'Electrical'),
('Provider Three', 'Mike Brown', '555-1003', 'provider3@example.com', '103 Provider St', 'HVAC'),
('Provider Four', 'Sara White', '555-1004', 'provider4@example.com', '104 Provider St', 'Lift'),
('Provider Five', 'Paul Black', '555-1005', 'provider5@example.com', '105 Provider St', 'Internet'),
('Provider Six', 'Nancy Green', '555-1006', 'provider6@example.com', '106 Provider St', 'Garbage Collection'),
('Provider Seven', 'Karen Blue', '555-1007', 'provider7@example.com', '107 Provider St', 'Pest Control'),
('Provider Eight', 'Frank Gray', '555-1008', 'provider8@example.com', '108 Provider St', 'Structural'),
('Provider Nine', 'Olivia Red', '555-1009', 'provider9@example.com', '109 Provider St', 'Security'),
('Provider Ten', 'Liam Yellow', '555-1010', 'provider10@example.com', '110 Provider St', 'Fire Safety'),
('Provider Eleven', 'Emma Purple', '555-1011', 'provider11@example.com', '111 Provider St', 'Landscaping'),
('Provider Twelve', 'Noah Orange', '555-1012', 'provider12@example.com', '112 Provider St', 'Painting'),
('Provider Thirteen', 'Ava Pink', '555-1013', 'provider13@example.com', '113 Provider St', 'Flooring'),
('Provider Fourteen', 'Sophia Gold', '555-1014', 'provider14@example.com', '114 Provider St', 'Roofing'),
('Provider Fifteen', 'Isabella Silver', '555-1015', 'provider15@example.com', '115 Provider St', 'Window Installation'),
('Provider Sixteen', 'Mason Bronze', '555-1016', 'provider16@example.com', '116 Provider St', 'Door Repairs'),
('Provider Seventeen', 'Logan Copper', '555-1017', 'provider17@example.com', '117 Provider St', 'Parking Solutions'),
('Provider Eighteen', 'Lucas Steel', '555-1018', 'provider18@example.com', '118 Provider St', 'Appliance Repair'),
('Provider Nineteen', 'Amelia Iron', '555-1019', 'provider19@example.com', '119 Provider St', 'Noise Management'),
('Provider Twenty', 'Olivia Brass', '555-1020', 'provider20@example.com', '120 Provider St', 'General Services');

-- *************************************************************
-- STEP 10: Insert Sample Data for Tickets (20 rows)
-- Note: For variety, odd-numbered rows have no assigned support agent;
-- even-numbered rows get assigned (assigned_to = (user_id mod 20)+1).
-- Rows where the row number is a multiple of 3 get an external provider.
-- *************************************************************
INSERT INTO Tickets (user_id, property_id, category_id, description, priority, sla_due, status_id, assigned_to, provider_id) VALUES
(1, 1, 1, 'Ticket 1: Plumbing issue reported', 'Low', DATE_ADD(NOW(), INTERVAL 48 HOUR), 1, NULL, NULL),
(2, 2, 2, 'Ticket 2: Lift issue reported', 'Medium', DATE_ADD(NOW(), INTERVAL 72 HOUR), 2, 3, NULL),
(3, 3, 3, 'Ticket 3: Electrical issue reported', 'High', DATE_ADD(NOW(), INTERVAL 24 HOUR), 3, NULL, 3),
(4, 4, 4, 'Ticket 4: HVAC issue reported', 'Urgent', DATE_ADD(NOW(), INTERVAL 36 HOUR), 4, 5, NULL),
(5, 5, 5, 'Ticket 5: Internet issue reported', 'Low', DATE_ADD(NOW(), INTERVAL 48 HOUR), 5, NULL, NULL),
(6, 6, 6, 'Ticket 6: Garbage collection issue reported', 'Medium', DATE_ADD(NOW(), INTERVAL 72 HOUR), 6, 7, 6),
(7, 7, 7, 'Ticket 7: Pest control issue reported', 'High', DATE_ADD(NOW(), INTERVAL 24 HOUR), 7, NULL, NULL),
(8, 8, 8, 'Ticket 8: Structural issue reported', 'Urgent', DATE_ADD(NOW(), INTERVAL 36 HOUR), 8, 9, NULL),
(9, 9, 9, 'Ticket 9: Security issue reported', 'Low', DATE_ADD(NOW(), INTERVAL 48 HOUR), 9, NULL, 9),
(10, 10, 10, 'Ticket 10: Fire safety issue reported', 'Medium', DATE_ADD(NOW(), INTERVAL 72 HOUR), 10, 11, NULL),
(11, 11, 11, 'Ticket 11: Landscaping issue reported', 'High', DATE_ADD(NOW(), INTERVAL 24 HOUR), 11, NULL, NULL),
(12, 12, 12, 'Ticket 12: Painting issue reported', 'Urgent', DATE_ADD(NOW(), INTERVAL 36 HOUR), 12, 13, 12),
(13, 13, 13, 'Ticket 13: Flooring issue reported', 'Low', DATE_ADD(NOW(), INTERVAL 48 HOUR), 13, NULL, NULL),
(14, 14, 14, 'Ticket 14: Roofing issue reported', 'Medium', DATE_ADD(NOW(), INTERVAL 72 HOUR), 14, 15, NULL),
(15, 15, 15, 'Ticket 15: Window issue reported', 'High', DATE_ADD(NOW(), INTERVAL 24 HOUR), 15, NULL, 15),
(16, 16, 16, 'Ticket 16: Door issue reported', 'Urgent', DATE_ADD(NOW(), INTERVAL 36 HOUR), 16, 17, NULL),
(17, 17, 17, 'Ticket 17: Parking issue reported', 'Low', DATE_ADD(NOW(), INTERVAL 48 HOUR), 17, NULL, NULL),
(18, 18, 18, 'Ticket 18: Appliance issue reported', 'Medium', DATE_ADD(NOW(), INTERVAL 72 HOUR), 18, 19, 18),
(19, 19, 19, 'Ticket 19: Noise complaint reported', 'High', DATE_ADD(NOW(), INTERVAL 24 HOUR), 19, NULL, NULL),
(20, 20, 20, 'Ticket 20: Special issue reported', 'Urgent', DATE_ADD(NOW(), INTERVAL 36 HOUR), 20, 1, NULL);

-- *************************************************************
-- STEP 11: Insert Sample Data for Ticket_Resolutions (20 rows)
-- Note: Each resolution corresponds to a ticket (ticket_id 1 to 20)
-- *************************************************************
INSERT INTO Ticket_Resolutions (ticket_id, resolution_description) VALUES
(1, 'Resolution: Fixed plumbing leak'),
(2, 'Resolution: Repaired elevator issue'),
(3, 'Resolution: Restored electrical wiring'),
(4, 'Resolution: Adjusted HVAC settings'),
(5, 'Resolution: Resolved internet outage'),
(6, 'Resolution: Cleared garbage collection issue'),
(7, 'Resolution: Addressed pest control concern'),
(8, 'Resolution: Repaired structural damage'),
(9, 'Resolution: Upgraded security system'),
(10, 'Resolution: Enhanced fire safety measures'),
(11, 'Resolution: Improved landscaping maintenance'),
(12, 'Resolution: Repainted affected areas'),
(13, 'Resolution: Repaired flooring damages'),
(14, 'Resolution: Fixed roof leak'),
(15, 'Resolution: Replaced broken windows'),
(16, 'Resolution: Repaired door mechanisms'),
(17, 'Resolution: Resolved parking management issue'),
(18, 'Resolution: Repaired malfunctioning appliance'),
(19, 'Resolution: Mitigated noise disturbance'),
(20, 'Resolution: Special issue resolved successfully');

-- *************************************************************
-- STEP 12: Insert Sample Data for Comments (20 rows)
-- Note: Comments reference an existing ticket_id (1-20) and a user_id (1-20)
-- *************************************************************
INSERT INTO Comments (ticket_id, user_id, comment_text) VALUES
(1, 2, 'Comment on ticket 1: Issue acknowledged.'),
(2, 3, 'Comment on ticket 2: Technician assigned.'),
(3, 4, 'Comment on ticket 3: Awaiting parts delivery.'),
(4, 5, 'Comment on ticket 4: Scheduled inspection.'),
(5, 6, 'Comment on ticket 5: Issue under review.'),
(6, 7, 'Comment on ticket 6: Replacement parts ordered.'),
(7, 8, 'Comment on ticket 7: Follow-up required.'),
(8, 9, 'Comment on ticket 8: Site visited by maintenance.'),
(9, 10, 'Comment on ticket 9: Security notified.'),
(10, 11, 'Comment on ticket 10: Fire safety check complete.'),
(11, 12, 'Comment on ticket 11: Landscaping adjustment made.'),
(12, 13, 'Comment on ticket 12: Painting touch-up scheduled.'),
(13, 14, 'Comment on ticket 13: Flooring evaluated.'),
(14, 15, 'Comment on ticket 14: Roofing repair planned.'),
(15, 16, 'Comment on ticket 15: Windows order placed.'),
(16, 17, 'Comment on ticket 16: Door mechanics reviewed.'),
(17, 18, 'Comment on ticket 17: Parking options discussed.'),
(18, 19, 'Comment on ticket 18: Appliance repair in progress.'),
(19, 20, 'Comment on ticket 19: Noise level measured.'),
(20, 1, 'Comment on ticket 20: Special instructions received.');

-- *************************************************************
-- STEP 13: Insert Sample Data for Attachments (20 rows)
-- *************************************************************
INSERT INTO Attachments (ticket_id, file_name, file_path) VALUES
(1, 'attachment1.jpg', '/files/attachment1.jpg'),
(2, 'attachment2.jpg', '/files/attachment2.jpg'),
(3, 'attachment3.jpg', '/files/attachment3.jpg'),
(4, 'attachment4.jpg', '/files/attachment4.jpg'),
(5, 'attachment5.jpg', '/files/attachment5.jpg'),
(6, 'attachment6.jpg', '/files/attachment6.jpg'),
(7, 'attachment7.jpg', '/files/attachment7.jpg'),
(8, 'attachment8.jpg', '/files/attachment8.jpg'),
(9, 'attachment9.jpg', '/files/attachment9.jpg'),
(10, 'attachment10.jpg', '/files/attachment10.jpg'),
(11, 'attachment11.jpg', '/files/attachment11.jpg'),
(12, 'attachment12.jpg', '/files/attachment12.jpg'),
(13, 'attachment13.jpg', '/files/attachment13.jpg'),
(14, 'attachment14.jpg', '/files/attachment14.jpg'),
(15, 'attachment15.jpg', '/files/attachment15.jpg'),
(16, 'attachment16.jpg', '/files/attachment16.jpg'),
(17, 'attachment17.jpg', '/files/attachment17.jpg'),
(18, 'attachment18.jpg', '/files/attachment18.jpg'),
(19, 'attachment19.jpg', '/files/attachment19.jpg'),
(20, 'attachment20.jpg', '/files/attachment20.jpg');

-- *************************************************************
-- STEP 14: Insert Sample Data for Audit_Log (20 rows)
-- Note: Logs changes made to tickets. References ticket_id (1-20) and user_id (changed_by).
-- *************************************************************
INSERT INTO Audit_Log (ticket_id, changed_by, change_description) VALUES
(1, 2, 'Audit: Ticket 1 updated by user 2.'),
(2, 3, 'Audit: Ticket 2 status changed.'),
(3, 4, 'Audit: Ticket 3 assigned to technician.'),
(4, 5, 'Audit: Ticket 4 updated with SLA changes.'),
(5, 6, 'Audit: Ticket 5 priority updated.'),
(6, 7, 'Audit: Ticket 6 details modified.'),
(7, 8, 'Audit: Ticket 7 resolved.'),
(8, 9, 'Audit: Ticket 8 escalated.'),
(9, 10, 'Audit: Ticket 9 reviewed by management.'),
(10, 11, 'Audit: Ticket 10 noted for follow-up.'),
(11, 12, 'Audit: Ticket 11 changes approved.'),
(12, 13, 'Audit: Ticket 12 adjustment recorded.'),
(13, 14, 'Audit: Ticket 13 updated for maintenance.'),
(14, 15, 'Audit: Ticket 14 updated by supervisor.'),
(15, 16, 'Audit: Ticket 15 changes implemented.'),
(16, 17, 'Audit: Ticket 16 status updated to On Hold.'),
(17, 18, 'Audit: Ticket 17 details revised.'),
(18, 19, 'Audit: Ticket 18 escalated for review.'),
(19, 20, 'Audit: Ticket 19 archived.'),
(20, 1, 'Audit: Ticket 20 special instructions updated.');

-- *************************************************************
-- STEP 15: Insert Sample Data for Notification_Log (20 rows)
-- Note: Each notification relates to a user and (optionally) a ticket.
-- *************************************************************
INSERT INTO Notification_Log (user_id, ticket_id, notification_message, status) VALUES
(1, 1, 'Notification: Ticket 1 has been created.', 'Sent'),
(2, 2, 'Notification: Ticket 2 has been assigned.', 'Sent'),
(3, 3, 'Notification: Ticket 3 status updated.', 'Pending'),
(4, 4, 'Notification: Ticket 4 requires attention.', 'Failed'),
(5, 5, 'Notification: Ticket 5 resolved.', 'Sent'),
(6, 6, 'Notification: Ticket 6 escalated.', 'Sent'),
(7, 7, 'Notification: Ticket 7 new update.', 'Pending'),
(8, 8, 'Notification: Ticket 8 rescheduled.', 'Sent'),
(9, 9, 'Notification: Ticket 9 under review.', 'Sent'),
(10, 10, 'Notification: Ticket 10 action required.', 'Failed'),
(11, 11, 'Notification: Ticket 11 has been updated.', 'Sent'),
(12, 12, 'Notification: Ticket 12 resolution in progress.', 'Sent'),
(13, 13, 'Notification: Ticket 13 closed.', 'Sent'),
(14, 14, 'Notification: Ticket 14 archived.', 'Sent'),
(15, 15, 'Notification: Ticket 15 reopened.', 'Pending'),
(16, 16, 'Notification: Ticket 16 update received.', 'Sent'),
(17, 17, 'Notification: Ticket 17 requires inspection.', 'Sent'),
(18, 18, 'Notification: Ticket 18 assigned.', 'Failed'),
(19, 19, 'Notification: Ticket 19 completed.', 'Sent'),
(20, 20, 'Notification: Ticket 20 special update.', 'Sent');




-- ======================================================
-- DROP procedures, triggers and functions if they exist
-- (Optional: uncomment if needed to rebuild)
-- ======================================================
/*
DROP PROCEDURE IF EXISTS spCreateTicket;
DROP PROCEDURE IF EXISTS spUpdateTicketStatus;
DROP PROCEDURE IF EXISTS spAssignTicket;
DROP PROCEDURE IF EXISTS spEscalateTicket;
DROP PROCEDURE IF EXISTS spCloseTicket;
DROP TRIGGER IF EXISTS tr_before_insert_ticket;
DROP TRIGGER IF EXISTS tr_after_update_ticket;
DROP TRIGGER IF EXISTS tr_after_insert_resolution;
DROP FUNCTION IF EXISTS f_getTicketAge;
DROP FUNCTION IF EXISTS f_getTicketPriority;
DROP FUNCTION IF EXISTS f_getUserFullName;
DROP FUNCTION IF EXISTS f_getIssueCategory;
DROP FUNCTION IF EXISTS f_timeRemainingForSLA;
*/

-- ======================================================
-- Stored Procedure 1: spCreateTicket
-- Creates a new ticket record in Tickets table.
-- Returns the new ticket's id via LAST_INSERT_ID().
-- ======================================================
DELIMITER $$
CREATE PROCEDURE spCreateTicket (
    IN p_user_id INT,
    IN p_property_id INT,
    IN p_category_id INT,
    IN p_description TEXT,
    IN p_priority VARCHAR(10),
    IN p_sla_due DATETIME,
    IN p_status_id INT,
    IN p_assigned_to INT,
    IN p_provider_id INT
)
BEGIN
    INSERT INTO Tickets (
        user_id, property_id, category_id, description, priority, sla_due, status_id, assigned_to, provider_id
    )
    VALUES (
        p_user_id, p_property_id, p_category_id, p_description, p_priority, p_sla_due, p_status_id, p_assigned_to, p_provider_id
    );
END$$
DELIMITER ;

-- ======================================================
-- Stored Procedure 2: spUpdateTicketStatus
-- Updates the status of a ticket and logs the change in Audit_Log.
-- ======================================================
DELIMITER $$
CREATE PROCEDURE spUpdateTicketStatus (
    IN p_ticket_id INT,
    IN p_status_id INT,
    IN p_changed_by INT
)
BEGIN
    -- Update the ticket status.
    UPDATE Tickets
    SET status_id = p_status_id, updated_at = NOW()
    WHERE ticket_id = p_ticket_id;

    -- Insert an audit log record.
    INSERT INTO Audit_Log(ticket_id, changed_by, change_description)
    VALUES (
        p_ticket_id, 
        p_changed_by, 
        CONCAT('Status updated to ', p_status_id, ' by user ', p_changed_by)
    );
END$$
DELIMITER ;

-- ======================================================
-- Stored Procedure 3: spAssignTicket
-- Assigns a ticket to a support agent.
-- ======================================================
DELIMITER $$
CREATE PROCEDURE spAssignTicket (
    IN p_ticket_id INT,
    IN p_assigned_to INT,
    IN p_changed_by INT
)
BEGIN
    UPDATE Tickets
    SET assigned_to = p_assigned_to, updated_at = NOW()
    WHERE ticket_id = p_ticket_id;

    INSERT INTO Audit_Log(ticket_id, changed_by, change_description)
    VALUES (
        p_ticket_id,
        p_changed_by,
        CONCAT('Ticket assigned to user ', p_assigned_to)
    );
END$$
DELIMITER ;

-- ======================================================
-- Stored Procedure 4: spEscalateTicket
-- Escalates a ticket's priority:
--   'Low' -> 'Medium', 'Medium' -> 'High', 'High' -> 'Urgent'
-- If already at 'Urgent' then no change.
-- ======================================================
DELIMITER $$
CREATE PROCEDURE spEscalateTicket (
    IN p_ticket_id INT,
    IN p_changed_by INT
)
BEGIN
    DECLARE current_priority VARCHAR(10);

    SELECT priority INTO current_priority
    FROM Tickets
    WHERE ticket_id = p_ticket_id;

    IF current_priority = 'Low' THEN
        UPDATE Tickets SET priority = 'Medium', updated_at = NOW() WHERE ticket_id = p_ticket_id;
    ELSEIF current_priority = 'Medium' THEN
        UPDATE Tickets SET priority = 'High', updated_at = NOW() WHERE ticket_id = p_ticket_id;
    ELSEIF current_priority = 'High' THEN
        UPDATE Tickets SET priority = 'Urgent', updated_at = NOW() WHERE ticket_id = p_ticket_id;
    ELSE
        -- If already 'Urgent', no escalation.
        UPDATE Tickets SET priority = 'Urgent', updated_at = NOW() WHERE ticket_id = p_ticket_id;
    END IF;

    INSERT INTO Audit_Log(ticket_id, changed_by, change_description)
    VALUES (
        p_ticket_id,
        p_changed_by,
        CONCAT('Ticket escalated from ', current_priority, ' to ', (SELECT priority FROM Tickets WHERE ticket_id = p_ticket_id))
    );
END$$
DELIMITER ;

-- ======================================================
-- Stored Procedure 5: spCloseTicket
-- Closes a ticket by updating its status (to "Closed") and logs a resolution.
-- Note: Assumes that the Ticket_Status table has a row with status_name = 'Closed'.
-- ======================================================
DELIMITER $$
CREATE PROCEDURE spCloseTicket (
    IN p_ticket_id INT,
    IN p_resolution_description TEXT,
    IN p_changed_by INT
)
BEGIN
    DECLARE v_closed_status INT;

    -- Retrieve the status id for "Closed"
    SELECT status_id INTO v_closed_status
    FROM Ticket_Status
    WHERE status_name = 'Closed'
    LIMIT 1;

    -- Update the ticket's status to "Closed"
    UPDATE Tickets
    SET status_id = v_closed_status, updated_at = NOW()
    WHERE ticket_id = p_ticket_id;

    -- Insert into Ticket_Resolutions
    INSERT INTO Ticket_Resolutions(ticket_id, resolution_description)
    VALUES (p_ticket_id, p_resolution_description);

    INSERT INTO Audit_Log(ticket_id, changed_by, change_description)
    VALUES (
        p_ticket_id,
        p_changed_by,
        'Ticket closed and resolution logged.'
    );
END$$
DELIMITER ;

-- ======================================================
-- TRIGGER 1: tr_before_insert_ticket
-- Ensures that sla_due is later than created_at upon ticket insertion.
-- ======================================================
DELIMITER $$
CREATE TRIGGER tr_before_insert_ticket
BEFORE INSERT ON Tickets
FOR EACH ROW
BEGIN
    IF NEW.sla_due < NEW.created_at THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'SLA due date must be after creation date';
    END IF;
END$$
DELIMITER ;

-- ======================================================
-- TRIGGER 2: tr_after_update_ticket
-- After a ticket is updated, log an audit record capturing old and new priorities.
-- ======================================================
DELIMITER $$
CREATE TRIGGER tr_after_update_ticket
AFTER UPDATE ON Tickets
FOR EACH ROW
BEGIN
    IF OLD.priority <> NEW.priority THEN
        INSERT INTO Audit_Log(ticket_id, changed_by, change_description)
        VALUES (
            NEW.ticket_id,
            COALESCE(NEW.assigned_to, 0),
            CONCAT('Ticket priority changed from ', OLD.priority, ' to ', NEW.priority)
        );
    END IF;
END$$
DELIMITER ;

-- ======================================================
-- TRIGGER 3: tr_after_insert_resolution
-- After inserting a ticket resolution, update the corresponding ticket's status to Closed.
-- ======================================================
DELIMITER $$
CREATE TRIGGER tr_after_insert_resolution
AFTER INSERT ON Ticket_Resolutions
FOR EACH ROW
BEGIN
    UPDATE Tickets 
    SET status_id = (
        SELECT status_id FROM Ticket_Status WHERE status_name = 'Closed' LIMIT 1
    )
    WHERE ticket_id = NEW.ticket_id;
END$$
DELIMITER ;

-- ======================================================
-- FUNCTION 1: f_getTicketAge
-- Returns the age of a ticket (in hours) from creation to now.
-- ======================================================
DELIMITER $$
CREATE FUNCTION f_getTicketAge(p_ticket_id INT)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE v_created DATETIME;
    DECLARE v_hours INT;

    SELECT created_at INTO v_created
    FROM Tickets
    WHERE ticket_id = p_ticket_id;

    SET v_hours = TIMESTAMPDIFF(HOUR, v_created, NOW());
    RETURN v_hours;
END$$
DELIMITER ;

-- ======================================================
-- FUNCTION 2: f_getTicketPriority
-- Returns the priority value of a specific ticket.
-- ======================================================
DELIMITER $$
CREATE FUNCTION f_getTicketPriority(p_ticket_id INT)
RETURNS VARCHAR(10)
DETERMINISTIC
BEGIN
    DECLARE v_priority VARCHAR(10);

    SELECT priority INTO v_priority
    FROM Tickets
    WHERE ticket_id = p_ticket_id;

    RETURN v_priority;
END$$
DELIMITER ;

-- ======================================================
-- FUNCTION 3: f_getUserFullName
-- Returns the full name of a user given the user_id.
-- ======================================================
DELIMITER $$
CREATE FUNCTION f_getUserFullName(p_user_id INT)
RETURNS VARCHAR(100)
DETERMINISTIC
BEGIN
    DECLARE v_name VARCHAR(100);

    SELECT full_name INTO v_name
    FROM Users
    WHERE user_id = p_user_id;

    RETURN v_name;
END$$
DELIMITER ;

-- ======================================================
-- FUNCTION 4: f_getIssueCategory
-- Returns the issue category name given a category_id.
-- ======================================================
DELIMITER $$
CREATE FUNCTION f_getIssueCategory(p_category_id INT)
RETURNS VARCHAR(50)
DETERMINISTIC
BEGIN
    DECLARE v_category VARCHAR(50);

    SELECT category_name INTO v_category
    FROM Issue_Categories
    WHERE category_id = p_category_id;

    RETURN v_category;
END$$
DELIMITER ;

-- ======================================================
-- FUNCTION 5: f_timeRemainingForSLA
-- Returns the remaining time (in hours) for a ticket's SLA.
-- A negative value indicates the SLA is overdue.
-- ======================================================
DELIMITER $$
CREATE FUNCTION f_timeRemainingForSLA(p_ticket_id INT)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE v_sla DATETIME;
    DECLARE v_remaining INT;

    SELECT sla_due INTO v_sla
    FROM Tickets
    WHERE ticket_id = p_ticket_id;

    SET v_remaining = TIMESTAMPDIFF(HOUR, NOW(), v_sla);
    RETURN v_remaining;
END$$
DELIMITER ;

-- ------------------------------------------------------
-- Query 1 – Detailed Ticket Report
-- ------------------------------------------------------
-- This query retrieves comprehensive details for each ticket.
-- It joins multiple tables to show the ticket description, 
-- the user who raised it, property details, issue category, 
-- current status, assigned support agent (if any), and 
-- nicely formatted dates.
SELECT 
    t.ticket_id,                           -- Unique ticket identifier
    t.description,                         -- Ticket description
    u.full_name AS raised_by,              -- Name of the user who raised the ticket
    p.property_name,                       -- Name of the associated property
    ic.category_name,                      -- Issue category name (e.g., Plumbing Issue)
    ts.status_name,                        -- Current status (e.g., Open, Closed)
    sa.full_name AS assigned_to,           -- Support agent's name (if assigned)
    DATE_FORMAT(t.created_at, '%Y-%m-%d %H:%i') AS created_at,  -- Formatted creation date
    DATE_FORMAT(t.sla_due, '%Y-%m-%d %H:%i') AS sla_due         -- Formatted SLA due date
FROM Tickets t
LEFT JOIN Users u ON t.user_id = u.user_id
LEFT JOIN Properties p ON t.property_id = p.property_id
LEFT JOIN Issue_Categories ic ON t.category_id = ic.category_id
LEFT JOIN Ticket_Status ts ON t.status_id = ts.status_id
LEFT JOIN Users sa ON t.assigned_to = sa.user_id
ORDER BY t.ticket_id;

-- ------------------------------------------------------
-- Query 2 – Ticket Count Per Property with Manager Details
-- ------------------------------------------------------
-- This query counts how many tickets have been raised for each property.
-- It shows the property name and the manager's name by joining the Properties table with Tickets
-- and the Users table (to get manager details).
SELECT 
    p.property_name,                        -- Property name
    u.full_name AS manager,                 -- Manager's name for the property
    COUNT(t.ticket_id) AS ticket_count      -- Total number of tickets associated with the property
FROM Properties p
LEFT JOIN Tickets t ON p.property_id = t.property_id
LEFT JOIN Users u ON p.manager_id = u.user_id
GROUP BY p.property_id, p.property_name, u.full_name   -- Group results by property and manager
ORDER BY ticket_count DESC;

-- ------------------------------------------------------
-- Query 3 – Tickets Nearing SLA Breach
-- ------------------------------------------------------
-- This query shows tickets that have less than 12 hours remaining until their SLA deadline.
-- It uses the custom function f_timeRemainingForSLA to calculate the hours remaining.
SELECT 
    t.ticket_id,                                     -- Ticket identifier
    t.description,                                   -- Ticket description
    f_timeRemainingForSLA(t.ticket_id) AS hours_remaining,  -- Hours remaining until SLA deadline
    ts.status_name,                                  -- Current ticket status
    DATE_FORMAT(t.sla_due, '%Y-%m-%d %H:%i') AS sla_due -- Formatted SLA due date
FROM Tickets t
LEFT JOIN Ticket_Status ts ON t.status_id = ts.status_id
WHERE f_timeRemainingForSLA(t.ticket_id) < 12       -- Filter for tickets nearing SLA breach
ORDER BY f_timeRemainingForSLA(t.ticket_id) ASC;

-- ------------------------------------------------------
-- Query 4 – Average Resolution Time per Issue Category
-- ------------------------------------------------------
-- This query calculates the average resolution time (in hours) for resolved tickets
-- in each issue category. It joins Tickets with Ticket_Resolutions and groups results 
-- by issue category.
SELECT 
    ic.category_name,  -- Issue category name
    AVG(TIMESTAMPDIFF(HOUR, t.created_at, tr.resolved_at)) AS avg_resolution_time_hours,  
                       -- Average resolution time (in hours)
    COUNT(t.ticket_id) AS resolved_tickets  -- Number of resolved tickets
FROM Tickets t
JOIN Ticket_Resolutions tr ON t.ticket_id = tr.ticket_id  -- Only include resolved tickets
JOIN Issue_Categories ic ON t.category_id = ic.category_id  -- Group results by issue category
GROUP BY ic.category_id, ic.category_name
ORDER BY avg_resolution_time_hours;

-- ------------------------------------------------------
-- Query 5 – Audit History for a Specific Ticket
-- ------------------------------------------------------
-- This query retrieves all audit log entries for a specific ticket (e.g., ticket_id = 1).
-- It also joins the Users table to display the full name of the user who made each change.
SELECT 
    al.ticket_id,                         -- Ticket identifier from audit log
    al.changed_by,                        -- User ID who made the change
    u.full_name AS changed_by_name,       -- Full name of the user who made the change
    al.change_description,                -- Description of the change made
    al.change_timestamp                   -- Timestamp when the change was recorded
FROM Audit_Log al
LEFT JOIN Users u ON al.changed_by = u.user_id  -- Join to include user's full name
WHERE al.ticket_id = 1                    -- Filter for a specific ticket (ticket_id = 1)
ORDER BY al.change_timestamp;

-- ------------------------------------------------------
-- Query 6 – Tickets Count by Status
-- ------------------------------------------------------
-- This query aggregates and counts the number of tickets for each current status
-- (e.g., Open, In Progress, Closed) by joining with the Ticket_Status table.
SELECT 
    ts.status_name,                     -- Current status name
    COUNT(t.ticket_id) AS ticket_count  -- Number of tickets in that status
FROM Tickets t
JOIN Ticket_Status ts ON t.status_id = ts.status_id  -- Join to get descriptive status names
GROUP BY ts.status_name              -- Group results by status
ORDER BY ticket_count DESC;

-- ------------------------------------------------------
-- Query 7 – Comments with Ticket and User Details
-- ------------------------------------------------------
-- This query lists all comments along with details from the associated ticket
-- and the user who posted the comment, providing context for each comment.
SELECT 
    c.comment_id,                                   -- Unique identifier for the comment
    c.ticket_id,                                    -- Associated ticket identifier
    t.description AS ticket_description,            -- Ticket description for context
    c.user_id,                                      -- ID of the user who commented
    u.full_name,                                    -- Full name of the user who commented
    c.comment_text,                                 -- The text of the comment
    DATE_FORMAT(c.created_at, '%Y-%m-%d %H:%i') AS comment_date  -- Formatted comment creation date
FROM Comments c
LEFT JOIN Tickets t ON c.ticket_id = t.ticket_id   -- Join to get ticket details
LEFT JOIN Users u ON c.user_id = u.user_id           -- Join to get user details
ORDER BY c.created_at;

-- ------------------------------------------------------
-- Query 8 – SLA Performance per Issue Category
-- ------------------------------------------------------
-- This query calculates, for each issue category, the percentage of resolved tickets 
-- that were closed within the SLA timeframe. It compares actual resolution time with the SLA.
SELECT 
    ic.category_name,  -- Issue category name
    COUNT(t.ticket_id) AS total_resolved,  -- Total number of resolved tickets in this category
    SUM(CASE 
          WHEN TIMESTAMPDIFF(HOUR, t.created_at, tr.resolved_at) <= TIMESTAMPDIFF(HOUR, t.created_at, t.sla_due) 
          THEN 1 
          ELSE 0 
        END) AS resolved_within_sla,  -- Count of tickets resolved on time
    ROUND( (SUM(CASE 
                 WHEN TIMESTAMPDIFF(HOUR, t.created_at, tr.resolved_at) <= TIMESTAMPDIFF(HOUR, t.created_at, t.sla_due) 
                 THEN 1 ELSE 0 END) / COUNT(t.ticket_id)) * 100, 2) AS percent_within_sla  
                       -- Percentage of tickets resolved within SLA
FROM Tickets t
JOIN Ticket_Resolutions tr ON t.ticket_id = tr.ticket_id  -- Only consider tickets with a resolution
JOIN Issue_Categories ic ON t.category_id = ic.category_id
GROUP BY ic.category_name
ORDER BY percent_within_sla DESC;

-- ------------------------------------------------------
-- Query 9 – User Ticket Summary
-- ------------------------------------------------------
-- This query provides a summary for each user, showing the total number of tickets 
-- they've raised, how many have been closed, and their average resolution time for closed tickets.
SELECT 
    u.user_id,                             -- User identifier
    u.full_name,                           -- User's full name
    COUNT(t.ticket_id) AS total_tickets,   -- Total tickets raised by the user
    SUM(CASE WHEN ts.status_name = 'Closed' THEN 1 ELSE 0 END) AS closed_tickets,  
                                           -- Number of closed tickets
    AVG(CASE 
          WHEN ts.status_name = 'Closed' 
          THEN TIMESTAMPDIFF(HOUR, t.created_at, tr.resolved_at) 
          ELSE NULL 
        END) AS avg_resolution_time_hours  -- Average resolution time for closed tickets (in hours)
FROM Users u
LEFT JOIN Tickets t ON u.user_id = t.user_id
LEFT JOIN Ticket_Status ts ON t.status_id = ts.status_id
LEFT JOIN Ticket_Resolutions tr ON t.ticket_id = tr.ticket_id
GROUP BY u.user_id, u.full_name
ORDER BY total_tickets DESC;

-- ------------------------------------------------------
-- Query 10 – Monthly Ticket Trend
-- ------------------------------------------------------
-- This query aggregates the number of tickets created each month to analyze trends over time.
SELECT 
    DATE_FORMAT(t.created_at, '%Y-%m') AS month,  -- Group by year-month (e.g., 2025-03)
    COUNT(t.ticket_id) AS tickets_created         -- Total tickets created during the month
FROM Tickets t
GROUP BY DATE_FORMAT(t.created_at, '%Y-%m')
ORDER BY month;                                   -- Order results chronologically
