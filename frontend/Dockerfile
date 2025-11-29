FROM nginx:alpine
 
# Copy Angular build output
COPY  dist/hrms/browser/ /usr/share/nginx/html
 
# Set permissions
RUN chmod -R 755 /usr/share/nginx/html
 
# Copy Nginx config
COPY default.conf /etc/nginx/conf.d/default.conf
 
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
 
