"# Week10" 
Video: https://drive.google.com/file/d/1hFaK6R-qQWj8yfR0AdmgobAckZ_PmTsm/view?usp=sharing




package Project7;
import java.math.BigDecimal;
import java.util.List;
import java.util.Objects;
import java.util.Scanner;
import Project7.entity.Project;
import Project7.exception.DbException;
import Project7.service.ProjectsService;
public class project7 {
	private Scanner scanner = new Scanner(System.in);
	private ProjectsService projectsService = new ProjectsService();
	private Project curProject;
	
	private List<String> operations = List.of(
			
			"1) create tables",
			"2) Add projects",
			"3) Select a project"
	);
	
	public static void main(String[] args) {
		new project7().displayMenu();
		
	}
	
		
	private void displayMenu() {
		// TODO Auto-generated method stub
		boolean done = false;
		
		while(!done) {
			int operation = getOperation();
			
			try {
		
			switch(operation) {
			case -1:
				done = exitMenu();
				break;
				
			case 1:
				createTables();
				break;
			case 2:
				addProject();
				break;
			case 3:
				selectProject();
				break;
				
				default:
					System.out.println("\n" + operation + " is not valid. Try again.");
					break;
					
			
			}
			
		}
			catch (Exception e) {
			System.out.println("\nError " + e.toString() + " Try again.");
		}
		}
	}
	
	private void selectProject() {
		listProject();
		Integer projectId = getIntInput("Enter a project ID to select a project");
		
		curProject = null;
		
		curProject = projectsService.fetchProjectById(projectId);
if(Objects.isNull(curProject)) {
	System.out.println("\n you are not working with a project.");
}
else {
	System.out.println("\nYou are working with project: " + curProject);
}
	}
	private void listProject() {
		List<Project> projects = projectsService.fetchALLproject();
		
		System.out.println("\nProject:");
		
		projects.forEach(project -> System.out.println(" " + project.getProject_id() + ": " + project.getProjectsName()));
		
		
	}
	private void addProject() {
		String name = getStringInput( "Enter the project name");
		String notes = getStringInput("Enter the project notes");
		BigDecimal actualHours = getDecimalInput("Enter number the actual hours");
		Integer difficulty = getIntInput("Enter the project dificulty(1-5)");
		BigDecimal estimatedHours = getDecimalInput("Enter estimated hours");
		
		
		
		Project project = new Project();
		
		project.setProjectsName(name);
		project.setNotes(notes);
		project.setDifficulty(difficulty);
		project.setEstimatedHours(estimatedHours);
		project.setActualHours(actualHours);
		
		
		Project dbProject = projectsService.addProject(project);
		System.out.println("You added this project:\n" + dbProject);
		
	}
	private BigDecimal getDecimalInput(String prompt) {
		//Need to create to complete
		String input = getStringInput(prompt);
		
		if(Objects.isNull(input)) {
	
		return null;
		}
		try {
			return new BigDecimal(input).setScale(2);
			
		}
		catch(NumberFormatException e) {
			throw new DbException(input + " is not a valid decimal number. ");
		}
		}
	private void createTables() {
	projectsService.createAndPopulateTables();
	System.out.println("\nTables created and populated.");
		
	}
	private boolean exitMenu() {
	System.out.println("\nExisiting the menu.");
		return true;
	}
	private int getOperation() {
	printOperations();
	
	Integer op = getIntInput("\nEnter an operation number (press Enter to quit");
	
	return Objects.isNull(op) ? -1 : op;
	}
	private Integer getIntInput(String prompt) {
		String input = getStringInput(prompt);
		
		if (Objects.isNull(input)) {
			return null;
		}
		try {
			return Integer.parseInt(input);
		}	
		catch(NumberFormatException e) {
			throw new DbException(input + " is not a valid number.");
		}
	}
	
	private Double getDoubleInput(String prompt) {
		String input = getStringInput(prompt);
		
		if (Objects.isNull(input)) {
			return null;
		}
		try {
			return Double.parseDouble(input);
			
		}	
		catch(NumberFormatException e) {
			throw new DbException(input + " is not a valid number.");
		}
			
		}
	private String getStringInput(String prompt) {
		System.out.print(prompt + ": ");
		String line = scanner.nextLine();
		
		return line.isBlank() ? null : line.trim();
	}
	
	private void printOperations() {
		System.out.println();
		System.out.println("Heres what you can do:");
		
		
		operations.forEach(op -> System.out.println(" " + op));;
		
	}
}
	

package Project7.service;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.LinkedList;
import java.util.List;
import java.util.NoSuchElementException;
import Project7.dao.ProjectsDao;
import Project7.entity.Project;
import Project7.exception.DbException;
public class ProjectsService {
	
private static final String SCHEMA_FILE = "projects_schema.sql";
private static final String DATA_FILE = "projects_data.sql";
	
private ProjectsDao ProjectsDAO = new ProjectsDao();
public Project fetchProjectById(Integer projectId) {
	return ProjectsDAO.fetchProjectById(projectId).orElseThrow(() -> new NoSuchElementException("Project with project ID=" + projectId + " does not exist."));
}
	
	public void createAndPopulateTables() {
		loadFromFile(SCHEMA_FILE);
		loadFromFile (DATA_FILE);
		
		
		
	}
	private void loadFromFile(String fileName) {
		String content = readFileContent (fileName);
		List<String> sqlStatements = convertContentToSqlStatements(content);
		
		
		
		
	}
	private List<String> convertContentToSqlStatements(String content) {
		content = removeComments(content);
		content = replaceWhitespaceSequencesWithSingleSpace(content);
		
		return extractLinesFromContent(content);
		
	}
	private List<String> extractLinesFromContent(String content) {
		List<String> lines = new LinkedList<>();
		
		while(!content.isEmpty()) {
			int semicolon = content.indexOf(";");
			if (semicolon == -1) {
				if(!content.isBlank()) {
					lines.add(content);
					
				}
				content = "";
			}
			else {
				lines.add(content.substring(0 , semicolon).trim());
				content = content.substring(semicolon + 1);
			}
		}
		return lines;
		
	}
	
	
	private String replaceWhitespaceSequencesWithSingleSpace(String content) {
		
		return content.replaceAll("\\s+", " ");
	}
	private String removeComments(String content) {
		StringBuilder builder = new StringBuilder(content);
		int commentPos = 0;
		
		while((commentPos = builder.indexOf("-- ", commentPos)) != -1) {
			int eolPos = builder.indexOf("\n", commentPos +1);
			
			if(eolPos == -1) {
				builder.replace(commentPos, builder.length(), "");
				
			}
			else {
				builder.replace(commentPos, eolPos + 1, "");
			}
			
		}
		
		return builder.toString();
	}
	private String readFileContent(String fileName) {
		try {
		
		Path path = Paths.get(getClass().getClassLoader().getResource(fileName).toURI());
		return Files.readString(path);
		
	} catch(Exception e) {
		
		throw new DbException(e);
	}
	
	}
	public static void main(String[] args) {
		new ProjectsService().createAndPopulateTables();
	}
	public Project addProject(Project project) {
	
		return ProjectsDAO.insertProject(project);
	}
	public List<Project> fetchALLproject() {
		
		return ProjectsDAO.fetchAllProjects();
	}
}

package Project7.dao;
import java.math.BigDecimal;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.LinkedList;
import java.util.List;
import Project7.entity.Category;
import java.util.Objects;
import java.util.Optional;
import Project7.entity.Materials;
import Project7.entity.Project;
import Project7.entity.Step;
import Project7.exception.DbException;
import provided.util.DaoBase;
/**
* @author Jose
*
*/
public class ProjectsDao extends DaoBase {
	
	private static final String MATERIAL_TABLE = "material";
	private static final String PROJECT_TABLE = "project";
	private static final String STEPS_TABLE = "step";
	private static final String CATEGORY_TABLE = "category";
	private static final String PROJECT_CATEGORY_TABLE = "project_category";
	
	public List<Project> fetchAllProjects() {
		String sql = "SLECT * FROM " + PROJECT_TABLE + " ORDER BY project_name";
		
		try(Connection conn = DbConnection.getConnection()) {
			startTransaction(conn);
			
		try(PreparedStatement stmt = conn.prepareStatement(sql)) {
			try(ResultSet rs = stmt.executeQuery()) {
				List<Project> projects = new LinkedList<>();
				
			while(rs.next()) {
				projects.add(extract(rs, Project.class));
				
			}
			return projects;
			}
		}
		catch(Exception e) {
			rollbackTransaction(conn);
			throw new DbException(e);
		}
		}
		catch (SQLException e) {
			throw new DbException(e);		
			}
	}
	
	public Optional<Project> fetchProjectById(Integer projectId) {
		String sql = "SELECT * FROM " + PROJECT_TABLE + " WHERE project_id = ?";
		
		try(Connection conn = DbConnection.getConnection()) {
			startTransaction(conn);
		
		try {
			Project project = null;
	
			
			try(PreparedStatement stmt = conn.prepareStatement(sql)) {
				setParameter(stmt, 1, projectId, Integer.class);
				
				try(ResultSet rs = stmt.executeQuery()) {
					if(rs.next()) {
						project = extract(rs, Project.class);
					}
				}
			}
		
			if(Objects.nonNull(project)) {
				project.getMaterial().addAll(fetchMaterialsForProject(conn, projectId));
				project.getSteps().addAll(fetchStepsForProject(conn, projectId));
				project.getCategories().addAll(fetchCategoriesForProject(conn, projectId));
			
			}
			commitTransaction(conn);
			return Optional.ofNullable(project);
			
		}
			catch(Exception e) {
				rollbackTransaction(conn);
				throw new DbException(e);
			}
		
		}
		catch (SQLException e) {
			throw new DbException(e);
		}
	}
	private List<Category> fetchCategoriesForProject(Connection conn, Integer projectId) throws SQLException {
		
		String sql = ""
				
				+ "SELECT c. * FROM " + CATEGORY_TABLE + " c "
				+ "JOIN " + PROJECT_CATEGORY_TABLE + " pc USING (category_id) "
				+ "WHERE project_id = ?";
		
		try(PreparedStatement stmt = conn.prepareStatement(sql)) {
			setParameter(stmt, 1, projectId, Integer.class);
			
			try(ResultSet rs = stmt.executeQuery()) {
				List<Category> categories= new LinkedList<>();
				
				while(rs.next()) {
					categories.add(extract(rs, Category.class));
				
				}
				
				return categories;
			}
		}
		
	}
	private List<Step> fetchStepsForProject(Connection conn, Integer projectId)
		throws SQLException {
			String sql = "SELECT * FROM " + STEPS_TABLE + " WHERE project_id = ?";
			
			try(PreparedStatement stmt = conn.prepareStatement(sql)) {
				setParameter(stmt, 1, projectId, Integer.class);
				
				try(ResultSet rs = stmt.executeQuery()) {
					List<Step> steps = new LinkedList<>();
					
					while(rs.next()) {
						steps.add(extract(rs, Step.class));
						
					}
					
					return steps;
					
				}
			}
			
			
			
		}
		
		
		
		
	private List<Materials> fetchMaterialsForProject(Connection conn, Integer projectId)
	throws SQLException {
		String sql = "SELECT * FROM " + MATERIAL_TABLE + " WHERE project_id = ?";
		
		try(PreparedStatement stmt = conn.prepareStatement(sql)) {
			setParameter(stmt, 1, projectId, Integer.class);
			
			try(ResultSet rs = stmt.executeQuery()) {
				List<Materials> materials = new LinkedList<>();
				
				while(rs.next()) {
					materials.add(extract(rs, Materials.class));
				}
				
				return materials;
			}
		}
	}
	public Project insertProject(Project project) {
		String sql = ""
			+ "INSERT INTO " + PROJECT_TABLE + " "
			+ "(project_name, notes, difficulty, estimated_hours, actual_hours) "
			+ "VALUES"
			+ "(?, ?, ?, ?, ?)";
		
		try(Connection conn = DbConnection.getConnection()) {
			startTransaction(conn);
		try(PreparedStatement stmt = conn.prepareStatement(sql)) {
			setParameter(stmt, 1, project.getProjectsName(), String.class);
			setParameter(stmt, 2, project.getNotes(), String.class);
			setParameter(stmt, 3, project.getDifficulty(), Integer.class);
			setParameter(stmt, 4, project.getEstimatedHours(), BigDecimal.class);
			setParameter(stmt, 5, project.getActualHours(), BigDecimal.class);
			
			stmt.executeUpdate();
			Integer projectId = getLastInsertId(conn, PROJECT_TABLE);
			
			commitTransaction(conn);
			
			project.setProject_id(projectId);
			return project;
			
			
		}
		catch(Exception e) {
			rollbackTransaction(conn);
			throw new DbException(e);
		}
		} catch (Exception e) {
			throw new DbException(e);
			
		}
	}
	
public void executeBatch(List<String> sqlBatch) {
	try(Connection conn= DbConnection.getConnection()) {
		startTransaction(conn);
		
		try(Statement stmt = conn.createStatement()) {
			for(String sql : sqlBatch) {
				stmt.addBatch(sql);
			}
			stmt.executeBatch();
			commitTransaction(conn);
		}
		catch(Exception e) {
			rollbackTransaction(conn);
			throw new DbException(e);
		}
	
	} catch (SQLException e) {
	throw new DbException(e);
	
		
	}
	
}
}
	
	
	
	
