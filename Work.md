# procedure to add data in table
DELIMITER //

CREATE PROCEDURE sp_HireRanger (
    IN p_employee_id INT,
    IN p_employee_name VARCHAR(50),
    IN p_age INT,
    IN p_gender VARCHAR(10),
    IN p_assigned_zone VARCHAR(2), -- Changed to match zone_id
    IN p_experience_years INT,
    IN p_shift VARCHAR(20),
    IN p_role VARCHAR(50),
    IN p_category VARCHAR(30),
    OUT p_message VARCHAR(255)
)
BEGIN
    -- Error handler for duplicate employee_id
    DECLARE EXIT HANDLER FOR 1062 -- 1062 is the error code for duplicate primary key
    BEGIN
        SET p_message = 'Failed to hire: Employee ID already exists.';
        ROLLBACK;
    END;
    
    -- Error handler for foreign key violation
    DECLARE EXIT HANDLER FOR 1452 -- 1452 is for foreign key errors
    BEGIN
        SET p_message = 'Failed to hire: Assigned zone does not exist.';
        ROLLBACK;
    END;

    START TRANSACTION;
        INSERT INTO rangers_staff (
            employee_id, employee_name, age, gender, assigned_zone,
            experience_years, shift, role, category
        ) VALUES (
            p_employee_id, p_employee_name, p_age, p_gender, p_assigned_zone,
            p_experience_years, p_shift, p_role, p_category
        );
        SET p_message = 'Ranger hired successfully.';
    COMMIT;
END //

DELIMITER ;

# trigger before insert
DELIMITER //

CREATE TRIGGER trg_staff_before_insert
BEFORE INSERT ON rangers_staff
FOR EACH ROW
BEGIN
    -- Check Age
    IF NEW.age <= 18 THEN
        SIGNAL SQLSTATE '45000' -- '45000' is a generic error state
        SET MESSAGE_TEXT = 'Cannot add ranger: Age must be greater than 18.';
    END IF;

    -- Check Experience
    IF NEW.experience_years < 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cannot add ranger: Experience years cannot be negative.';
    END IF;

    -- Check Gender
    IF NEW.gender NOT IN ('Male', 'Female', 'Other') THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cannot add ranger: Gender must be ''Male'', ''Female'', or ''Other''.';
    END IF;
END //

DELIMITER ;

# trigger before update
CREATE TRIGGER trg_staff_before_update
BEFORE UPDATE ON rangers_staff
FOR EACH ROW
BEGIN
    -- Check Age
    IF NEW.age <= 18 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cannot update ranger: Age must be greater than 18.';
    END IF;

    -- Check Experience
    IF NEW.experience_years < 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cannot update ranger: Experience years cannot be negative.';
    END IF;

    -- Check Gender
    IF NEW.gender NOT IN ('Male', 'Female', 'Other') THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cannot update ranger: Gender must be ''Male'', ''Female'', or ''Other''.';
    END IF;
END //

DELIMITER ;

---

# procedure to add medical checkup data

DELIMITER //

CREATE PROCEDURE sp_AddCheckup (
    IN p_animal_id INT,
    IN p_checkup_date DATE,
    IN p_health_status VARCHAR(255),
    IN p_weight_kg DECIMAL(10, 2),
    IN p_vaccination_status VARCHAR(255),
    IN p_next_checkup_date DATE,
    OUT p_message VARCHAR(255)
)
BEGIN
    -- Error handler for foreign key violation
    -- (e.g., trying to add a checkup for an animal_id that doesn't exist)
    DECLARE EXIT HANDLER FOR 1452 
    BEGIN
        SET p_message = 'Failed to add checkup: Animal ID does not exist.';
        ROLLBACK;
    END;

    -- General error handler
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        SET p_message = 'An unknown error occurred.';
        ROLLBACK;
    END;

    -- Start transaction
    START TRANSACTION;
        
        INSERT INTO medical_checkups (
            animal_id,
            checkup_date,
            health_status,
            weight_kg,
            vaccination_status,
            next_checkup_date
        )
        VALUES (
            p_animal_id,
            p_checkup_date,
            p_health_status,
            p_weight_kg,
            p_vaccination_status,
            p_next_checkup_date
        );
        
        SET p_message = 'Medical checkup added successfully.';

    COMMIT;

END //

DELIMITER ;

---

# procedure to add animal to database
DELIMITER //

CREATE PROCEDURE sp_LogAnimalSurvey (
    IN p_species_id VARCHAR(255),
    IN p_name VARCHAR(255),
    IN p_status VARCHAR(255),
    IN p_count INT,
    IN p_habitat_zone VARCHAR(255),
    IN p_last_survey DATE,
    IN p_image_url VARCHAR(255)
)
BEGIN
    INSERT INTO animals (
        species_id, name, status, count, habitat_zone, last_survey, image_url
    )
    VALUES (
        p_species_id, p_name, p_status, p_count, p_habitat_zone, p_last_survey, p_image_url
    )
    ON DUPLICATE KEY UPDATE -- This is the magic!
        count = p_count,
        last_survey = p_last_survey,
        status = p_status,
        image_url = p_image_url;
END //

DELIMITER ;

---

# aggregate function
SELECT
    SUM(area) AS total_area,
    SUM(camera_traps) AS total_cameras
FROM zones;

# prodedure
DELIMITER //

CREATE PROCEDURE sp_AddZone_V2 (
    IN p_zone_id VARCHAR(2),
    IN p_zone_name VARCHAR(255),
    IN p_area DECIMAL(10, 2),
    IN p_climate VARCHAR(30),
    IN p_camera_traps INT,
    IN p_access_level VARCHAR(30),
    IN p_primary_species VARCHAR(200),
    OUT p_message VARCHAR(255)
)
BEGIN
    -- Declare a handler that will run if any SQL error (SQLEXCEPTION) occurs.
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
	
        ROLLBACK;
	
        SET p_message = 'Failed to add zone. An error occurred (e.g., duplicate name/ID).';
    END;

    -- Start a transaction block.
    START TRANSACTION;

        -- Perform the insertion.
        INSERT INTO zones (
            zone_id, zone_name, area, climate, camera_traps,
            access_level, primary_species
        )
        VALUES (
            p_zone_id, p_zone_name, p_area, p_climate, p_camera_traps,
            p_access_level, p_primary_species
        );

        -- If the INSERT was successful, set the success message.
        SET p_message = CONCAT('Successfully added zone: ', p_zone_name);

    -- If no errors occurred, commit the transaction to make the changes permanent.
    COMMIT;

END //

DELIMITER ;

# view
CREATE VIEW v_zones_enhanced AS
SELECT 
    z.zone_id,
    z.zone_name,
    z.area,
    z.climate,
    z.camera_traps,
    z.access_level,
    z.primary_species,
    -- Calculate camera density per square kilometer
    ROUND(z.camera_traps / z.area, 2) as camera_density,
    -- Count total bookings for this zone
    COALESCE(booking_stats.total_bookings, 0) as total_bookings,
    -- Count total visitors for this zone
    COALESCE(booking_stats.total_visitors, 0) as total_visitors,
    -- Calculate total revenue from this zone
    COALESCE(booking_stats.total_revenue, 0) as total_revenue,
    -- Get most recent booking date
    booking_stats.last_booking_date,
    -- Count animals in this zone
    COALESCE(animal_stats.animal_count, 0) as animal_count,
    -- Count species in this zone
    COALESCE(animal_stats.species_count, 0) as species_count
FROM zones z
LEFT JOIN (
    SELECT 
        safari_zone,
        COUNT(*) as total_bookings,
        SUM(person_count) as total_visitors,
        SUM(total_amount) as total_revenue,
        MAX(safari_date) as last_booking_date
    FROM tickets
    WHERE booking_status != 'cancelled'
    GROUP BY safari_zone
) booking_stats ON z.zone_id = booking_stats.safari_zone
LEFT JOIN (
    SELECT 
        habitat_zone,
        COUNT(*) as animal_count,
        COUNT(DISTINCT species_id) as species_count
    FROM animals
    GROUP BY habitat_zone
) animal_stats ON z.zone_id = animal_stats.habitat_zone;

---

-- Get all tickets booked by a specific visitor along with their name and email
SELECT
    v.name,
    v.email,
    t.booking_id,
    t.safari_date,
    t.time_slot,
    t.safari_zone,
    t.person_count,
    t.total_amount
FROM visitors v
JOIN tickets t ON v.id = t.visitor_id
WHERE v.email = '${email}';

# trigger for ticket
DELIMITER $$

CREATE TRIGGER before_ticket_insert_calculate_cost
BEFORE INSERT ON tickets
FOR EACH ROW
BEGIN
    DECLARE v_services_cost DECIMAL(10,2);
    DECLARE v_gst_rate DECIMAL(5,2) DEFAULT 0.18; -- 18% GST
    DECLARE v_guide_cost DECIMAL(10,2) DEFAULT 500.00;
    DECLARE v_camera_cost DECIMAL(10,2) DEFAULT 250.00;
    DECLARE v_lunch_cost DECIMAL(10,2) DEFAULT 300.00;
    DECLARE v_transport_cost DECIMAL(10,2) DEFAULT 1000.00;

    -- cost of additional services
    SET v_services_cost = 0;
    IF NEW.has_guide = 1 THEN SET v_services_cost = v_services_cost + v_guide_cost; END IF;
    IF NEW.has_camera = 1 THEN SET v_services_cost = v_services_cost + v_camera_cost; END IF;
    IF NEW.has_lunch = 1 THEN SET v_services_cost = v_services_cost + (v_lunch_cost * NEW.person_count); END IF;
    IF NEW.has_transport = 1 THEN SET v_services_cost = v_services_cost + v_transport_cost; END IF;

    SET NEW.services_cost = v_services_cost;
    SET NEW.gst_amount = (NEW.base_cost + v_services_cost) * v_gst_rate;
    SET NEW.total_amount = NEW.base_cost + v_services_cost + NEW.gst_amount;
END$$

DELIMITER ;



# procedure
DELIMITER $$

CREATE PROCEDURE BookSafariTicket(
    IN p_visitor_email VARCHAR(255),
    IN p_safari_date DATE,
    IN p_time_slot VARCHAR(20),
    IN p_safari_zone VARCHAR(20),
    IN p_person_count TINYINT,
    IN p_base_cost DECIMAL(10,2), 
    IN p_has_guide BOOLEAN,
    IN p_has_camera BOOLEAN,
    IN p_has_lunch BOOLEAN,
    IN p_has_transport BOOLEAN,

    OUT p_message VARCHAR(255)
)
BEGIN
    DECLARE v_visitor_id INT;
    DECLARE v_booking_id VARCHAR(25);
    DECLARE v_existing_booking_count INT DEFAULT 0;
    -- REMOVED: The hardcoded DECLARE v_base_cost line is gone.

    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET p_message = 'Error: Booking failed due to a database error. Transaction rolled back.';
    END;

    SELECT id INTO v_visitor_id FROM visitors WHERE email = p_visitor_email;

    IF v_visitor_id IS NULL THEN
        SET p_message = 'Error: Visitor not found. Please register first.';
    ELSE
        SELECT COUNT(*) INTO v_existing_booking_count FROM tickets
        WHERE visitor_id = v_visitor_id
          AND safari_date = p_safari_date
          AND time_slot = p_time_slot
          AND booking_status = 'confirmed';

        IF v_existing_booking_count > 0 THEN
            SET p_message = 'Error: You already have a confirmed booking for this date and time slot.';
        ELSE
            SET v_booking_id = CONCAT('BK', UNIX_TIMESTAMP(), v_visitor_id);

            START TRANSACTION;

            INSERT INTO tickets (
                booking_id, visitor_id, safari_date, time_slot, safari_zone,
                person_count, base_cost, has_guide, has_camera, has_lunch, has_transport
            )
            VALUES (
                v_booking_id, v_visitor_id, p_safari_date, p_time_slot, p_safari_zone,
                p_person_count, p_base_cost, -- CHANGED: Using the input parameter here
                p_has_guide, p_has_camera, p_has_lunch, p_has_transport
            );

            COMMIT;
            SET p_message = CONCAT('Success! Your booking ID is ', v_booking_id);
        END IF;
    END IF;
END$$

DELIMITER ;


# trigger for feedbacks
DELIMITER //

CREATE TRIGGER trg_Feedback_BeforeInsert
BEFORE INSERT ON feedbacks
FOR EACH ROW
BEGIN
    -- Check rating range
    IF NEW.rating_overall < 1 OR NEW.rating_overall > 5 THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'Overall rating must be between 1 and 5.';
    END IF;

    -- Enforce comments for bad reviews
    IF NEW.rating_overall <= 2 AND (NEW.comments IS NULL OR NEW.comments = '') THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Comments are required for ratings of 1 or 2.';
    END IF;
END //

DELIMITTER ;


# inner join on feedback and visitor
SELECT
    v.name,
    f.visit_date,
    f.booking_id,
    f.rating_overall,
    f.rating_guide,
    f.rating_facility,
    f.sightings,
    f.comments,
    f.recommend,
    f.submitted_at
FROM feedbacks f
INNER JOIN visitors v ON f.visitor_id = v.id
ORDER BY submitted_at DESC;

SELECT
    f.visit_date,
    f.booking_id,
    f.rating_overall,
    f.rating_guide,
    f.rating_facility,
    f.sightings,
    f.comments,
    f.recommend,
    f.submitted_at
FROM feedbacks f
INNER JOIN visitors v ON f.visitor_id = v.id
WHERE v.email = ?
ORDER BY f.submitted_at DESC;

---
