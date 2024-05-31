# practica5-login2

# Practica 5 Login (Parte 2)
En esta práctica continuaremos desarrollando un sistema de login utilizando Flask y MySQL. En esta parte, nos enfocaremos en enviar mensajes desde la ruta a la plantilla, y en la creación de la base de datos y los stored procedures necesarios.

# Enviando mensajes desde la ruta a la plantilla
Para enviar mensajes desde la ruta a la plantilla, utilizaremos la función flash de Flask. Para poder acceder a ella, debemos agregar una clave SECRET_KEY en la clase DevelopmentConfig del archivo config.py:


class DevelopmentConfig:
    DEBUG = True
    SECRET_KEY = "qhrf$edjYTJ)*21nsThdK"
# Luego, en la ruta /login, podemos agregar un mensaje de flash para indicar que el acceso fue rechazado:


if _user == "admin" and _pass == "123":
    return redirect(url_for("home"))
else:
    flash("Acceso rechazado...")
    return render_template("auth/login.html")
# Para mostrar estos mensajes en la plantilla login.html, podemos usar el siguiente bloque:


{% with messages = get_flashed_messages() %}
    {% if messages %}
        <ul>
            {% for message in messages %}
                <li class="alert alert-warning">{{ message }}</li>
            {% endfor %}
        </ul>
    {% endif %}
{% endwith %}
Base de datos
# Creación de la base de datos y tabla users
Para continuar con la práctica, necesitamos crear una base de datos llamada store y una tabla users con los campos username, password, fullname y usertype. El campo fullname puede ser nulo, mientras que el resto son obligatorios.

CREATE DATABASE store;
USE store;

CREATE TABLE users (
    id SMALLINT UNSIGNED NOT NULL AUTO_INCREMENT,
    username VARCHAR(20) NOT NULL,
    password CHAR(102) NOT NULL,
    fullname VARCHAR(50),
    usertype TINYINT NOT NULL,
    PRIMARY KEY (id)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
# Creación del stored procedure sp_AddUser
Este procedimiento almacenado servirá para dar de alta a un nuevo usuario en la base de datos. Utilizaremos la función SHA2 para codificar el password.

DELIMITER //
CREATE PROCEDURE sp_AddUser(IN pUserName VARCHAR(20), IN pPassword VARCHAR(102), IN pFullName VARCHAR(50), IN pUserType TINYINT)
BEGIN
    DECLARE hashedPassword VARCHAR(255);
    SET hashedPassword = SHA2(pPassword, 256);
    INSERT INTO users (username, password, fullname, usertype)
    VALUES (pUserName, hashedPassword, pFullName, pUserType);
END //
DELIMITER ;
# Creación del stored procedure sp_verifyIdentity
Este procedimiento recibe el valor de usuario y password y valida si son correctos.


DELIMITER //
CREATE PROCEDURE sp_verifyIdentity(IN pUsername VARCHAR(20), IN pPlainTextPassword VARCHAR(20))
BEGIN
    DECLARE storedPassword VARCHAR(255);

    SELECT password INTO storedPassword FROM users
    WHERE username = pUsername COLLATE utf8mb4_unicode_ci;

    IF storedPassword IS NOT NULL AND storedPassword = SHA2(pPlainTextPassword, 256) THEN
        SELECT id, username, storedPassword, fullname, usertype FROM users
        WHERE username = pUserName COLLATE utf8mb4_unicode_ci;
    ELSE
        SELECT NULL;
    END IF;
END //
DELIMITER ;
Estos son los pasos para continuar con la práctica de login en Flask con MySQL. Si tienes alguna pregunta o necesitas más información, ¡no dudes en preguntar!
