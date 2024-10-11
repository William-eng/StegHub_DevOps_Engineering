## Re-using Ansible artifacts
In Ansible, it's common for beginners to write a single large playbook file to handle their automation tasks. While this approach works, as your automation needs grow more complex, it becomes
beneficial to break your work into smaller, modular files. Organizing tasks into smaller, reusable components helps manage complexity and enhances efficiency. By distributing variables, tasks,
and plays across multiple files, you can reuse them in various playbooks for different use cases. For instance, if you need to update a customer database in several playbooks, placing these tasks
in a separate tasks file or role allows you to reuse the same code across different playbooks while maintaining it in one central location. This approach simplifies updates and reduces 
redundancy in your automation setup.



## Creating reusable files and roles
Ansible offers four distributed, reusable artifacts: variables files, task files, playbooks, and roles.

- A variables file contains only variables.

- A task file contains only tasks.

- A playbook contains at least one play, and may contain variables, tasks, and other content. You can reuse tightly focused playbooks, but you can only reuse them statically, not dynamically.

- A role contains a set of related tasks, variables, defaults, handlers, and even modules or other plugins in a defined file-tree. Unlike variables files, task files, or playbooks, roles can be easily uploaded and shared through Ansible Galaxy. See Roles for details about creating and using roles.


You can incorporate multiple playbooks into a main playbook. However, you can only use imports to reuse playbooks. For example:

    - import_playbook: webservers.yml
    - import_playbook: databases.yml
    
Importing incorporates playbooks in other playbooks statically. Ansible runs the plays and tasks in each imported playbook in the order they are listed, just as if they had been defined directly in the main playbook.

You can select which playbook you want to import at runtime by defining your imported playbook file name with a variable, then passing the variable with either _--extra-vars_ or the _vars_ keyword. For example:

      - import_playbook: "/path/to/{{ import_from_extra_var }}"
      - import_playbook: "{{ import_from_vars }}"
        vars:
          import_from_vars: /path/to/one_playbook.yml

If you run this playbook with _ansible-playbook my_playbook -e import_from_extra_var=other_playbook.yml_, Ansible imports both one_playbook.yml and other_playbook.yml.

## When to turn a playbook into a role
For some use cases, simple playbooks work well. However, starting at a certain level of complexity, roles work better than playbooks. A role lets you store your defaults, handlers, variables, and tasks in separate directories, instead of in a single long document. Roles are easy to share on Ansible Galaxy. For complex use cases, most users find roles easier to read, understand, and maintain than all-in-one playbooks.

## Re-using files and roles
Ansible offers two ways to reuse files and roles in a playbook: dynamic and static.

For dynamic reuse, add an _include_*_ task in the tasks section of a play:

- include_role

- include_tasks

- include_vars

For static reuse, add an _import_*_ task in the tasks section of a play:

- import_role

- import_tasks

Task include and import statements can be used at arbitrary depth.

You can still use the bare roles keyword at the play level to incorporate a role in a playbook statically. However, the bare include keyword, once used for both task files and playbook-level includes, is now deprecated.


##  Includes: dynamic reuse

Including roles, tasks, or variables in Ansible dynamically adds them to a playbook, allowing the included content to be processed in real-time as the playbook runs. 
The outcome of earlier tasks can influence whether included tasks or roles are executed, making them behave like handlers—they may or may not run based on prior results in the playbook. 
A key benefit of using `include_*` statements is the ability to use loops. When looping is applied to an inclusion, the tasks or roles are repeated for each item in the loop. File names for these includes are templated, allowing for flexibility, and variables can be passed into includes to adapt the playbook further. For more information on variable inheritance, check out Ansible's variable precedence documentation.

## Imports: static reuse
Importing roles, tasks, or playbooks adds them to a playbook statically, meaning Ansible processes these imports before running any tasks. As a result, imported content is not influenced by the results of other tasks in the top-level playbook. File names for imported roles and tasks support templating, but the required variables must be available during the pre-processing stage. This can be done using the `vars` keyword or through `--extra-vars`. You can also pass variables to imports, which is necessary if you need to run the same imported file multiple times within a playbook.




      tasks:
      - import_tasks: wordpress.yml
        vars:
          wp_user: timmy
      
      - import_tasks: wordpress.yml
        vars:
          wp_user: alice
      
      - import_tasks: wordpress.yml
        vars:
          wp_user: bob



















































































































