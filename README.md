# -_aplicacion_de_Calculo_Estadistico_java_- :.
Aplicacion de Calculo Estadistico:

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/0f7272d7-84a7-41d8-9dc8-706f66959636" />  

```
Java + IntelliJ IDEA + Swing + Oracle 19c
Desarrollaremos una aplicacion de escritorio en:
Java
IntelliJ IDEA
Swing (Interfaz gráfica)
Oracle Database 19c

La aplicación permitira:
Registrar conjuntos de datos numericos

Calcular:
Media
Mediana
Moda
Varianza
Desviación estándar
Máximo
Mínimo
Guardar resultados en Oracle 19c
Consultar historial de calculos .

Arquitectura del Proyecto
CalculadoraEstadistica/
│
├── src/
│   ├── conexion/
│   │     ConexionOracle.java
│   │
│   ├── modelo/
│   │     ResultadoEstadistico.java
│   │
│   ├── dao/
│   │     ResultadoDAO.java
│   │
│   ├── logica/
│   │     EstadisticaService.java
│   │
│   ├── vista/
│   │     FrmEstadistica.java
│   │
│   └── Main.java
│
└── ojdbc11.jar

1. Script Oracle 19c
Crear Tabla
CREATE TABLE RESULTADOS_ESTADISTICOS (
    ID NUMBER GENERATED ALWAYS AS IDENTITY,
    DATOS VARCHAR2(4000),
    MEDIA NUMBER(10,2),
    MEDIANA NUMBER(10,2),
    MODA NUMBER(10,2),
    VARIANZA NUMBER(10,2),
    DESVIACION NUMBER(10,2),
    MAXIMO NUMBER(10,2),
    MINIMO NUMBER(10,2),
    FECHA_REGISTRO DATE DEFAULT SYSDATE,
    CONSTRAINT PK_RESULTADOS PRIMARY KEY(ID)
);

2. Configurar Librería Oracle
Descargar
Oracle JDBC Driver
Producto relacionado
Oracle JDBC Driver
Sitio oficial
Oracle JDBC Downloads
Agregar
ojdbc11.jar
En IntelliJ
File → Project Structure → Libraries

3. Clase de Conexión Oracle
ConexionOracle.java
package conexion;

import java.sql.Connection;
import java.sql.DriverManager;

public class ConexionOracle {

    private static final String URL =
            "jdbc:oracle:thin:@localhost:1521:XE";

    private static final String USER = "SYSTEM";
    private static final String PASSWORD = "123456";

    public static Connection conectar() {

        try {

            Class.forName("oracle.jdbc.driver.OracleDriver");

            return DriverManager.getConnection(
                    URL,
                    USER,
                    PASSWORD
            );

        } catch (Exception e) {

            System.out.println(
                    "Error conexión: " + e.getMessage()
            );

            return null;
        }
    }
}

4. Modelo
ResultadoEstadistico.java
package modelo;

public class ResultadoEstadistico {

    private String datos;
    private double media;
    private double mediana;
    private double moda;
    private double varianza;
    private double desviacion;
    private double maximo;
    private double minimo;

    public String getDatos() {
        return datos;
    }

    public void setDatos(String datos) {
        this.datos = datos;
    }

    public double getMedia() {
        return media;
    }

    public void setMedia(double media) {
        this.media = media;
    }

    public double getMediana() {
        return mediana;
    }

    public void setMediana(double mediana) {
        this.mediana = mediana;
    }

    public double getModa() {
        return moda;
    }

    public void setModa(double moda) {
        this.moda = moda;
    }

    public double getVarianza() {
        return varianza;
    }

    public void setVarianza(double varianza) {
        this.varianza = varianza;
    }

    public double getDesviacion() {
        return desviacion;
    }

    public void setDesviacion(double desviacion) {
        this.desviacion = desviacion;
    }

    public double getMaximo() {
        return maximo;
    }

    public void setMaximo(double maximo) {
        this.maximo = maximo;
    }

    public double getMinimo() {
        return minimo;
    }

    public void setMinimo(double minimo) {
        this.minimo = minimo;
    }
}

5. Logica Estadistica
EstadisticaService.java
package logica;

import java.util.*;

public class EstadisticaService {

    public double calcularMedia(double[] datos) {

        double suma = 0;

        for (double d : datos) {
            suma += d;
        }

        return suma / datos.length;
    }

    public double calcularMediana(double[] datos) {

        Arrays.sort(datos);

        int n = datos.length;

        if (n % 2 == 0) {

            return (datos[n / 2 - 1]
                    + datos[n / 2]) / 2;

        } else {

            return datos[n / 2];
        }
    }

    public double calcularModa(double[] datos) {

        Map<Double, Integer> frecuencia =
                new HashMap<>();

        for (double d : datos) {

            frecuencia.put(
                    d,
                    frecuencia.getOrDefault(d, 0) + 1
            );
        }

        double moda = datos[0];
        int max = 0;

        for (Map.Entry<Double, Integer> e :
                frecuencia.entrySet()) {

            if (e.getValue() > max) {

                max = e.getValue();
                moda = e.getKey();
            }
        }

        return moda;
    }

    public double calcularVarianza(
            double[] datos,
            double media
    ) {

        double suma = 0;

        for (double d : datos) {

            suma += Math.pow(d - media, 2);
        }

        return suma / datos.length;
    }

    public double calcularDesviacion(
            double varianza
    ) {

        return Math.sqrt(varianza);
    }

    public double calcularMaximo(double[] datos) {

        return Arrays.stream(datos)
                .max()
                .getAsDouble();
    }

    public double calcularMinimo(double[] datos) {

        return Arrays.stream(datos)
                .min()
                .getAsDouble();
    }
}

6. DAO Oracle
ResultadoDAO.java
package dao;

import conexion.ConexionOracle;
import modelo.ResultadoEstadistico;

import java.sql.Connection;
import java.sql.PreparedStatement;

public class ResultadoDAO {

    public void guardar(ResultadoEstadistico r) {

        String sql =
                "INSERT INTO RESULTADOS_ESTADISTICOS " +
                "(DATOS, MEDIA, MEDIANA, MODA, " +
                "VARIANZA, DESVIACION, MAXIMO, MINIMO) " +
                "VALUES (?, ?, ?, ?, ?, ?, ?, ?)";

        try (
                Connection cn =
                        ConexionOracle.conectar();

                PreparedStatement ps =
                        cn.prepareStatement(sql)
        ) {

            ps.setString(1, r.getDatos());
            ps.setDouble(2, r.getMedia());
            ps.setDouble(3, r.getMediana());
            ps.setDouble(4, r.getModa());
            ps.setDouble(5, r.getVarianza());
            ps.setDouble(6, r.getDesviacion());
            ps.setDouble(7, r.getMaximo());
            ps.setDouble(8, r.getMinimo());

            ps.executeUpdate();

            System.out.println(
                    "Datos guardados en Oracle"
            );

        } catch (Exception e) {

            System.out.println(
                    "Error: " + e.getMessage()
            );
        }
    }
}

7. Interfaz Gráfica
FrmEstadistica.java
package vista;

import dao.ResultadoDAO;
import logica.EstadisticaService;
import modelo.ResultadoEstadistico;

import javax.swing.*;
import java.awt.*;

public class FrmEstadistica extends JFrame {

    JTextField txtDatos;

    JTextArea txtResultado;

    JButton btnCalcular;

    public FrmEstadistica() {

        setTitle("Calculadora Estadística");

        setSize(700, 500);

        setDefaultCloseOperation(EXIT_ON_CLOSE);

        setLocationRelativeTo(null);

        iniciarComponentes();
    }

    private void iniciarComponentes() {

        JPanel panel = new JPanel();

        panel.setLayout(null);

        JLabel lblDatos =
                new JLabel(
                        "Ingrese números separados por coma:"
                );

        lblDatos.setBounds(20, 20, 300, 30);

        panel.add(lblDatos);

        txtDatos = new JTextField();

        txtDatos.setBounds(20, 60, 640, 35);

        panel.add(txtDatos);

        btnCalcular =
                new JButton("Calcular");

        btnCalcular.setBounds(250, 120, 150, 40);

        panel.add(btnCalcular);

        txtResultado = new JTextArea();

        JScrollPane scroll =
                new JScrollPane(txtResultado);

        scroll.setBounds(20, 190, 640, 230);

        panel.add(scroll);

        add(panel);

        btnCalcular.addActionListener(e -> calcular());
    }

    private void calcular() {

        try {

            String texto = txtDatos.getText();

            String[] partes =
                    texto.split(",");

            double[] datos =
                    new double[partes.length];

            for (int i = 0; i < partes.length; i++) {

                datos[i] =
                        Double.parseDouble(
                                partes[i].trim()
                        );
            }

            EstadisticaService es =
                    new EstadisticaService();

            double media =
                    es.calcularMedia(datos);

            double mediana =
                    es.calcularMediana(datos);

            double moda =
                    es.calcularModa(datos);

            double varianza =
                    es.calcularVarianza(
                            datos,
                            media
                    );

            double desviacion =
                    es.calcularDesviacion(
                            varianza
                    );

            double maximo =
                    es.calcularMaximo(datos);

            double minimo =
                    es.calcularMinimo(datos);

            txtResultado.setText(
                    "MEDIA: " + media +
                    "\nMEDIANA: " + mediana +
                    "\nMODA: " + moda +
                    "\nVARIANZA: " + varianza +
                    "\nDESVIACIÓN: " + desviacion +
                    "\nMÁXIMO: " + maximo +
                    "\nMÍNIMO: " + minimo
            );

            ResultadoEstadistico r =
                    new ResultadoEstadistico();

            r.setDatos(texto);
            r.setMedia(media);
            r.setMediana(mediana);
            r.setModa(moda);
            r.setVarianza(varianza);
            r.setDesviacion(desviacion);
            r.setMaximo(maximo);
            r.setMinimo(minimo);

            ResultadoDAO dao =
                    new ResultadoDAO();

            dao.guardar(r);

            JOptionPane.showMessageDialog(
                    null,
                    "Cálculo realizado y guardado"
            );

        } catch (Exception ex) {

            JOptionPane.showMessageDialog(
                    null,
                    "Error en datos ingresados"
            );
        }
    }
}

8. Clase Principal
Main.java
import vista.FrmEstadistica;

public class Main {

    public static void main(String[] args) {

        new FrmEstadistica().setVisible(true);
    }
}

9. Ejemplo de Datos
10,20,30,40,50,60,70

10. Resultado Esperado
MEDIA: 40
MEDIANA: 40
MODA: 10
VARIANZA: 400
DESVIACIÓN: 20
MÁXIMO: 70
MINIMO: 10

11. Funcionalidades Adicionales Recomendadas
Puedes agregar:
Graficas estadisticas
Exportar PDF
Exportar Excel
Historial de cálculos
Login de usuarios
Reportes Oracle
Dashboard estadístico
Histogramas
Distribución normal

Tecnologias Utilizadas:
IntelliJ IDEA
Oracle Database 19c
Java Swing
Oracle Corporation
JetBrains 
:. . / .  
